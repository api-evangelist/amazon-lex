---
title: "Reinforcement fine-tuning with LLM-as-a-judge"
url: "https://aws.amazon.com/blogs/machine-learning/reinforcement-fine-tuning-with-llm-as-a-judge/"
date: "Thu, 30 Apr 2026 20:07:25 +0000"
author: "Hemanth Kumar Jayakumar"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p>Large language models (LLMs) now drive the most advanced conversational agents, creative tools, and decision-support systems. However, their raw output often contains inaccuracies, policy misalignments, or unhelpful phrasing—issues that undermine trust and limit real-world utility. <em>Reinforcement Fine‑Tuning (RFT)</em> has emerged as the preferred method to align these models efficiently, using <em>automated reward signals</em> to replace costly manual labeling.</p> 
<p>At the heart of modern RFT is reward functions. They’re built for each domain through verifiable reward functions that can score LLM generations through a piece of code (Reinforcement Learning with Verifiable Rewards or RLVR) or with LLM-as-a-judge, where a separate language model evaluates candidate responses to guide alignment (Reinforcement Learning with AI Feedback or RLAIF). Both these methods provide scores to the RL algorithm to nudge the model to solve the problem at hand. In this post, we take a deeper look at how RLAIF or RL with LLM-as-a-judge works with Amazon Nova models effectively.</p> 
<h2><strong>Why RFT with LLM‑as‑a-judge compared to generic RFT?</strong></h2> 
<p>Reinforcement Fine-Tuning can use any reward signal, straightforward hand‑crafted rules (RLVR), or an LLM that evaluates model outputs (LLM-as-a-judge or RLAIF). RLAIF makes alignment far more flexible and powerful, especially when reward signals are vague and hard to craft manually. Unlike generic RFT rewards that rely on blunt numeric scoring like substring matching, an LLM judge reasons across multiple dimensions—correctness, tone, safety, relevance—providing context-aware feedback that captures subtleties and domain-specific nuances without task-specific retraining. Additionally, LLM judges offer built-in explainability through rationales (for example, “Response A cites peer-reviewed studies”), providing diagnostics that accelerate iteration, pinpoint failure modes directly, and reduce hidden misalignments, something static reward functions can’t do.</p> 
<h2><strong>Implementing LLM-as-a-judge: Six critical steps</strong></h2> 
<p>This section covers the key steps involved in designing and deploying LLM-as-a-judge reward functions.</p> 
<h3><strong>Select the judge architecture</strong></h3> 
<p>The first critical decision is selecting your judge architecture. LLM-as-a-judge offers two primary evaluation modes: <em>Rubric-based (point- based) judging</em> and <em>Preference-based judging</em>, each suited to different alignment scenarios.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Criteria</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Rubric-based judging</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Preference-based judging</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Evaluation method</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Assigns a numeric score to a single response using predefined criteria</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Compares two candidate responses side-by-side and selects the superior one</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Quality measurement</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Absolute quality measurements</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Relative quality through direct comparison</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Preferred used when</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Clear, quantifiable evaluation dimensions exist (accuracy, completeness, safety compliance)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Policy model should explore freely without reference data restrictions</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Data requirements</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Only requires careful prompt engineering to align the model to reward specifications</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Requires at least one response sample for preference comparison</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Generalizability</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Better for out-of-distribution data, avoids data bias</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Depends on quality of reference responses</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Evaluation style</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Mirrors absolute scoring systems</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Mirrors natural human evaluation through comparison</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Recommended starting point</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Start here if preference data is unavailable and RLVR unsuitable</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Use when comparative data is available</td> 
  </tr> 
 </tbody> 
</table> 
<h3><strong>Define your evaluation criteria</strong></h3> 
<p>After you’ve selected your judge type, articulate the specific dimensions that you want to improve. Clear evaluation criteria are the foundation of effective RLAIF training.</p> 
<p><strong>For Preference-based judges:</strong></p> 
<p>Write clear prompts explaining what makes one response better than another. Be explicit about quality preferences with concrete examples. Example:&nbsp;<em>“Prefer responses that cite authoritative sources, use accessible language, and directly address the user’s question.”</em></p> 
<p><strong>For Rubric-based judges:</strong></p> 
<p>We recommend using&nbsp;Boolean (pass/fail) scoring&nbsp;for rubric-based judges. Boolean scoring is more reliable and reduces judge variability compared to fine-grained 1–10 scales. Define clear pass/fail criteria for each evaluation dimension with specific, observable characteristics.</p> 
<h3><strong>Select and configure your judge model</strong></h3> 
<p>Choose an LLM with sufficient reasoning capability to evaluate your target domain, configured through Amazon Bedrock and called using a reward AWS Lambda function. For common domains like math, coding, and conversational capabilities, smaller models can work well with careful prompt engineering.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Model tier</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Preferred for</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Cost</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Reliability</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Amazon</strong> <strong>Bedrock model</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Large/Heavyweight</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Complex reasoning, nuanced evaluation, multi-dimensional scoring</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">High</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Very High</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon Nova Pro, Claude Opus, Claude Sonnet</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Medium/Lightweight</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">General domains like math or coding, balanced cost-performance</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Low-Medium</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Moderate-High</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon Nova 2 Lite, Claude Haiku</td> 
  </tr> 
 </tbody> 
</table> 
<h3><strong>Refine your judge model prompt</strong></h3> 
<p>Your judge prompt is the foundation of alignment quality. Design it to produce structured, parseable outputs with clear scoring dimensions:</p> 
<ul> 
 <li><strong>Structured output format</strong>&nbsp;– Specify JSON or parseable format for straightforward extraction</li> 
 <li><strong>Clear scoring rules</strong>&nbsp;– Define exactly how each dimension should be calculated</li> 
 <li><strong>Edge case handling</strong>&nbsp;– Address ambiguous scenarios (for example, “If response is empty, assign score 0”)</li> 
 <li><strong>Desired behaviors</strong>&nbsp;– Explicitly state behaviors to encourage or discourage</li> 
</ul> 
<h3><strong>Align judge criteria with production evaluation metrics</strong></h3> 
<p>Your reward function should mirror the metrics that you will use to evaluate the final model in production. Align your reward function with production success criteria to enable models designed for the correct objectives.</p> 
<p><strong>Alignment workflow:</strong></p> 
<ol> 
 <li><strong>Define </strong>production success criteria (for example, accuracy, safety) with acceptable thresholds</li> 
 <li><strong>Map</strong> each criterion to specific judge scoring dimensions</li> 
 <li><strong>Validate</strong> that judge scores correlate with your evaluation metrics</li> 
 <li><strong>Test</strong> the judge on representative samples and edge cases</li> 
</ol> 
<h3><strong>Building </strong>a robust reward Lambda function</h3> 
<p>Production RFT systems process thousands of reward evaluations per training step. Build a resilient reward Lambda function to help provide training stability, efficient compute usage, and reliable model behavior. This section covers how to build a reward Lambda function that’s resilient, efficient, and production ready.</p> 
<p><strong>Composite reward score structuring</strong></p> 
<p>Don’t rely solely on LLM judges. Combine them with fast, deterministic reward components that catch obvious failures before expensive judge evals:</p> 
<p><strong>Core components</strong></p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Component</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Purpose</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>When to use</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Format correctness</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Verify JSON structure, required fields, schema compliance</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Always – catches malformed outputs immediately. Cheap and instant feedback.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Length penalties</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Discourage overly verbose or terse responses</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">When output length matters (for example, summaries)</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Language consistency</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Verify responses match input language</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Critical for multilingual applications</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Safety filters</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Rule-based checks for prohibited content</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Always – prevents unsafe content from reaching production</td> 
  </tr> 
 </tbody> 
</table> 
<p><strong>Infrastructure readiness</strong></p> 
<ol> 
 <li><strong>Implement exponential backoff:</strong> Handles Amazon Bedrock API rate limits and transient failures gracefully</li> 
 <li><strong>Parallelization strategy</strong>: Use ThreadPoolExecutor or async patterns to parallelize judge calls across rollouts to reduce latency</li> 
 <li><strong>Avoid Lambda cold start delays:</strong> Set an appropriate Lambda timeout (15 minutes recommended) and provisioned concurrency (~100 for typical setups)</li> 
 <li><strong>Error handling:</strong> Add comprehensive error handling that returns neutral/noisy rewards (0.5) rather than failing the entire training step</li> 
</ol> 
<p><strong>Test your reward Lambda function for resilience </strong></p> 
<p>Validate judge consistency and calibration:</p> 
<ul> 
 <li><strong>Consistency</strong>: Test judge on the same samples multiple times to measure score variance (should be low for deterministic evaluation)</li> 
 <li><strong>Cross-judge comparison: </strong>Compare scores across different judge models to identify evaluation blind spots</li> 
 <li><strong>Human calibration:</strong> Periodically sample rollouts for human review to catch judge drift or systematic errors</li> 
 <li><strong>Regression testing: </strong>Create a “judge test suite” with known good/bad examples to regression test judge behavior</li> 
</ul> 
<h2><strong>RFT with LLM-as-a-judge – Training workflow </strong></h2> 
<p>The following diagram illustrates the complete end-to-end training process, from baseline evaluation through judge validation to production deployment. Each step builds upon the previous one, creating a resilient pipeline that balances alignment quality with computational efficiency while actively preventing reward hacking and supporting production-ready model behavior.</p> 
<p><img alt="Five-stage AI model training and deployment pipeline diagram showing Setup, Training, and Deployment phases" class="aligncenter size-full wp-image-129427" height="607" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-1.png" width="1228" /></p> 
<h2><strong>Real-world case study: Automating legal contract review </strong></h2> 
<p>In this section, we refer to a real-world use case with a leading legal industry partner. The task is to generate comments on risks, assessments, and actions on legal documentation with respect to the policies and previous contracts as reference documents.</p> 
<h3><strong>Challenge</strong></h3> 
<p>Partner was interested in solving the problem of automating the process of reviewing, assessing, and flagging risks in legal contract documents. Specifically, they wanted to evaluate potential new contracts against internal guidelines and regulations, past contracts, and laws of the country pertaining to the contract.</p> 
<h3><strong>Solution </strong></h3> 
<p>We formulated this problem as one where we are providing a target document (the “contract” that needs evaluation), and a reference document (the grounding document and context) and expect the LLM to generate a JSON with multiple comments, comment types, and recommended actions to take based on the assessment. The original dataset available for this use case was relatively small that included complete contracts along with annotations and comments from legal experts. We used LLM as a judge using GPT OSS 120b model as the judge and a custom system prompt during RFT.</p> 
<h3><strong>RFT workflow </strong></h3> 
<p>In the following section we cover details of the key aspects in the RFT workflow for this use case.</p> 
<h4><strong>Reward Lambda function for LLM-as-a-judge</strong></h4> 
<p>The following code snippets present the key components of the reward Lambda function.</p> 
<p><strong>Note</strong>: name of Lambda function should have “SageMaker”, for example, <code>"arn:aws:lambda:us-east-1:123456789012:function:MyRewardFunction<strong>SageMaker</strong>"</code></p> 
<p><strong>a) Start with defining a high-level objective</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-json"># Contract Review Evaluation - Unweighted Scoring
You are an expert contract reviewer evaluating AI-generated comments. Your PRIMARY objective is to assess how well each predicted comment identifies issues in the TargetDocument contract clauses and whether those issues are justified by the Reference guidelines.</code></pre> 
</div> 
<p><strong>b) Define the evaluation approach </strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-txt">## Evaluation Approach
For each sample, you receive:
- **TargetDocument**: The contract text being reviewed (the document under evaluation)
- **Reference**: Reference guidelines/standards used for the review (the evaluation criteria)
- **Prediction**: One or more comments from the AI model
**Important**: The SystemPrompt shows what instructions the model received. Consider whether the model followed these instructions when evaluating the prediction quality.
**CRITICAL**: Each comment must identify a specific issue, gap, or concern IN THE TARGETDOCUMENT CONTRACT TEXT ITSELF. The comment's text_excerpt field should quote problematic contract language from the TargetDocument, NOT quote text from the Reference guidelines. The Reference justifies WHY the contract clause is problematic, but the issue must exist IN the contract.
Evaluate EACH predicted comment independently. Comments should flag problems in the contract clauses, not merely cite Reference requirements.</code></pre> 
</div> 
<p><strong>c) Describe the scoring dimensions with clear specifications on how a particular score should be calculated</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-txt">## Scoring Dimensions (Per Comment)
**EVALUATION ORDER**: Evaluate in this sequence: (1) TargetDocument_Grounding, (2) Reference_Consistency, (3) Actionability
### 1. TargetDocument_Grounding
**Evaluates**: (a) Whether text_excerpt quotes from TargetDocument contract text, and (b) Whether the comment is relevant to the quoted text_excerpt
**MANDATORY**: text_excerpt must quote from TargetDocument contract text. If text_excerpt quotes from Reference instead, score MUST be 1.
- **5**: text_excerpt correctly quotes TargetDocument contract text AND comment identifies a highly relevant, valid, and notable issue in that quoted text
- **4**: text_excerpt correctly quotes TargetDocument contract text AND comment identifies a valid and relevant issue in that quoted text
- **3**: text_excerpt correctly quotes TargetDocument contract text AND comment is somewhat relevant to that quoted text, but concern has moderate validity
- **2**: text_excerpt correctly quotes TargetDocument contract text BUT comment has weak relevance to that quoted text, or concern is questionable
- **1**: text_excerpt does NOT quote TargetDocument contract text (quotes Reference instead, or no actual quote), OR comment is irrelevant to the quoted text
### 2. Reference_Consistency
...
...</code></pre> 
</div> 
<p><strong>d) Clearly define the final output format to parse</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-txt">## Scoring Calculation
**Comment_Score** = Simple average of the three dimensions:
- Comment_Score = (TargetDocument_Grounding + Reference_Consistency + Actionability) / 3
**Aggregate_Score** = Average of all Comment_Score values for the sample
## Output Format
For each sample, evaluate ALL predicted comments and provide:
```json
{ "comments": [ 
        { "comment_id": "...",
          "TargetDocument_Grounding": {"score": X, "justification": "...", "supporting_evidence": "Verify text_excerpt quotes actual TargetDocument contract text and comment is relevant to it"},
          "Reference_Consistency": {"score": X, "justification": "...", "supporting_reference": "Quote from Reference that justifies the concern OR explain meaningful reasoning"},                   
          "Actionability": {"score": X, "justification": "Assess if action is clear, grounded in TargetDocument and Reference, and relevant to comment"},
          "Comment_Score": X.XX 
        } ],
  "Aggregate_Score": {
          "score": X.XX,
          "total_comments": N,
          "rationale": "..." 
   }
}
```</code></pre> 
</div> 
<p><strong>e) Create a high-level Lambda handler, providing sufficient multithreading for faster inference</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-python">def lambda_handler(event, context): 
        scores: List[RewardOutput] = []
        samples = event
        max_workers = len(samples)
        print(f"Evaluating {len(samples)} items with {max_workers} threads...")
        with ThreadPoolExecutor(max_workers=max_workers) as executor:
                futures = [executor.submit(judge_answer, sample) for sample in samples]
                scores = [future.result() for future in futures]
        print(f"Completed {len(scores)} evaluations")
        return [asdict(score) for score in scores]</code></pre> 
</div> 
<h4><strong>Deployment of the Lambda function</strong></h4> 
<p>We used the following AWS Identity and Access Management (IAM) permissions and settings in the Lambda function. The following configurations are required for reward Lambda functions. RFT training can fail if any of them are missing.</p> 
<p><strong>a) Permissions for Amazon SageMaker AI execution role</strong></p> 
<p>Your Amazon SageMaker AI execution role must have permission to invoke your Lambda function. Add this policy to your Amazon SageMaker AI execution role:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
&nbsp;&nbsp;&nbsp;&nbsp;"Version": "2012-10-17",
&nbsp;&nbsp;&nbsp;&nbsp;"Statement": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Effect": "Allow",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Action": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"lambda:InvokeFunction"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Resource": "arn:aws:lambda:region:account-id:function:function-name"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;]
}</code></pre> 
</div> 
<p><strong>b) Permissions for Lambda function execution role</strong></p> 
<p>Your Lambda function’s execution role needs basic Lambda execution permissions and the permissions to Invoke the judge Amazon Bedrock model.</p> 
<p><strong>Note:</strong> This solution follows the AWS shared responsibility model. AWS is responsible for securing the infrastructure that runs AWS services in the cloud. You are responsible for securing your Lambda function code, configuring IAM permissions, implementing encryption and access controls, managing data security and privacy, configuring monitoring and logging, and verifying compliance with applicable regulations. Follow the principle of least privilege by scoping permissions to specific resource ARNs. For more information, see <a href="https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html" rel="noopener noreferrer" target="_blank">Security</a> in AWS Lambda and Amazon SageMaker AI <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/security.html" rel="noopener noreferrer" target="_blank">Security</a> in the AWS documentation.</p> 
<p><img alt="AWS IAM console showing role permissions with AWSLambdaBasicExecutionRole and BedrockAccess policies attached" class="aligncenter size-full wp-image-129429" height="433" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-3.png" width="975" /></p> 
<p><strong>c) Add provisioned concurrency </strong></p> 
<p>Publish a version of the Lambda and to enable the function to scale without fluctuations in latency, we added some provisioned concurrency.&nbsp;100 was sufficient in this case, however, there’s more room for cost improvements here.</p> 
<p><img alt="AWS Lambda versions management panel showing 10 published versions, with versions 27 and 28 listed on page 1" class="aligncenter size-full wp-image-129430" height="197" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-4.png" width="1431" /></p> 
<p><strong>d) Set Lambda timeout to 15 mins</strong></p> 
<p><img alt="AWS Lambda general configuration panel showing 128 MB memory, 512 MB ephemeral storage, and 15-minute timeout" class="aligncenter wp-image-129431" height="157" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-5.png" width="983" /></p> 
<h4><strong>Customizing the training configuration</strong></h4> 
<p>We launched Nova Forge SDK that can be used for the entire model customization lifecycle—from data preparation to deployment and monitoring. Nova Forge SDK removes the need to search for the appropriate recipes or container URI for specific techniques.</p> 
<p>You can use the Nova Forge SDK to customize training parameters in two ways: provide a full recipe YAML using recipe_path or pass specific fields using overrides for selective changes. For this use case, we use overrides to tune the rollout and trainer settings as shown in the following section.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># Launch training with recipe overrides
result = customizer.train(
        job_name="my-rft-run",
        rft_lambda_arn="&lt;your-lambda-arn&gt;",
        overrides={
                # Training config
                "max_length": 64000,
                "global_batch_size": 64,
                "reasoning_effort": None,
                # Data
                "shuffle": False,
                # Rollout
                "type": "off_policy_async",
                "age_tolerance": 2,
                "proc_num": 6,
                "number_generation": 8,
                "max_new_tokens": 16000,
                "set_random_seed": True,
                "temperature": 1,
                "top_k": 0,
                "lambda_concurrency_limit": 100,
                # Trainer
                "max_steps": 516,
                "save_steps": 32,
                "save_top_k": 17,
                "refit_freq": 4,
                "clip_ratio_high": 0.28,
                "ent_coeff": 0.0,
                "loss_scale": 1,
        },
)</code></pre> 
</div> 
<h4><strong>Results</strong></h4> 
<p>RFT with Amazon Nova 2 Lite achieved a 4.33 aggregate score—the highest performance across all evaluated models—while maintaining perfect JSON schema validation. This represents a significant improvement, demonstrating that RFT can produce production-ready, specialized models that outperform larger general-purpose alternatives.</p> 
<p>We evaluated models using a <strong>“best of k” single-comment setting,</strong> where each model generated multiple comments per sample and we scored the highest-quality output. This approach establishes an upper bound on performance and enables a fair comparison between models that produce single versus multiple outputs.</p> 
<p><img alt="Horizontal bar chart comparing relative performance scores of five AI models, with Nova 2.0-lite (RFT) and Nova 2.0-lite (SFT) tied at the top score of 1.00" class="wp-image-129440" height="378" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-10.png" width="942" /></p> 
<p style="text-align: center;">Figure 1 — JSON Schema Validation Scores (0–1 scale, higher is better)</p> 
<p><img alt="Horizontal bar chart comparing absolute performance scores of five AI models, with Nova 2.0-lite (RFT) scoring highest at 4.33 out of 5.00" class="aligncenter size-large wp-image-129441" height="378" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-11-1024x406.png" width="942" /></p> 
<p style="text-align: center;">Figure 2 — Aggregate LLM judge scores (1–5 scale, higher is better)</p> 
<h2><strong>Key takeaways:</strong></h2> 
<ol> 
 <li><strong>RFT achieved the highest performance among evaluated models in this study.</strong></li> 
</ol> 
<p>Amazon Nova 2 Lite with RFT achieved a <strong>4.33 aggregate</strong> score, outperforming both Claude Sonnet 4.5 and Claude Haiku 4.5, while also achieving perfect <strong>JSON schema validation. </strong></p> 
<ol start="2"> 
 <li><strong>Removes unnecessary training artifacts</strong></li> 
</ol> 
<p>During SFT iterations, we observed problematic behaviors including repetitive comment generation and unnatural Unicode character predictions. These issues, likely caused by overfitting or dataset imbalances, didn’t appear in RFT checkpoints. RFT’s reward-based improvements naturally discourages such artifacts, <strong>producing more robust and reliable outputs</strong>.</p> 
<ol start="3"> 
 <li><strong>Strong generalization to new judge criteria </strong></li> 
</ol> 
<p>When we evaluated RFT models using a modified judge prompt (aligned but not identical to the training reward function), performance remained strong. This demonstrates that RFT learns generalizable quality patterns rather than overfitting specific evaluation criteria. This is a critical advantage for real-world deployment where requirements evolve.</p> 
<ol start="4"> 
 <li><strong>Compute considerations</strong></li> 
</ol> 
<p>RFT required 4–8 rollouts per training sample, increasing compute costs compared to SFT. This overhead is amplified when using non-zero reasoning effort settings. However, for mission-critical applications where alignment quality directly impacts business outcomes—such as legal contract review, financial compliance, or healthcare documentation, the performance gains justify the additional compute costs.</p> 
<h2><strong>Conclusion</strong></h2> 
<p>Reinforcement Fine-Tuning (RFT) with LLM-as-a-judge represents a powerful approach to aligning LLMs for domain-specific applications. As demonstrated in our legal contract review case study, this methodology delivers significant improvements over both base models and traditional supervised fine-tuning (SFT) approaches, with RFT achieving the highest aggregate scores across all evaluation dimensions.&nbsp;For teams building mission-critical AI systems where alignment quality directly impacts business outcomes, RFT with LLM-as-a-judge offers a compelling path forward. The methodology’s explainability, flexibility, and superior performance make it particularly valuable for complex domains like legal review (or Financial Services or Healthcare) where subtle nuances matter.</p> 
<p>Organizations considering this approach should start small—validate their judge design on curated benchmarks, verify infrastructure resilience, and scale gradually while monitoring for reward hacking. With proper implementation, RFT can transform capable base models into highly specialized, production-ready systems that consistently deliver aligned, trustworthy outputs.</p> 
<p><strong><em>References</em></strong>:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/nova/latest/nova2-userguide/nova-forge-sdk.html" rel="noopener noreferrer" target="_blank">Amazon Nova Developer Guide for Amazon Nova 2</a></li> 
 <li><a href="https://github.com/aws/nova-forge-sdk" rel="noopener noreferrer" target="_blank">Nova Forge SDK- GitHub</a></li> 
 <li><a href="https://docs.aws.amazon.com/nova/latest/nova2-userguide/nova-reinforcement-fine-tuning.html" rel="noopener noreferrer" target="_blank">Reinforcement Fine-Tuning (RFT) with Amazon Nova models</a></li> 
</ol> 
<p><strong><em>Disclaimer: </em></strong></p> 
<p>The legal contract review use case described in this post is for technical demonstration purposes only. AI-generated contract analysis is not a substitute for professional legal advice. Consult qualified legal counsel for legal matters.</p> 
<hr style="width: 80%;" /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-129436" height="98" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-6-1.png" width="100" />
  </div> 
  <p><strong>Hemanth Kumar Jayakumar</strong> is an Applied Scientist at Amazon AGI, where he works on reinforcement learning and foundation models. He translates the latest ML research into scalable solutions, unlocking domain specialization of foundation models for customers. Outside of work, Hemanth enjoys traveling and hiking.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-129437" height="134" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-7.png" width="100" />
  </div> 
  <p><strong>Daniel Suarez Souto</strong> is a Solutions Architect at Amazon Web Services, specializing in Artificial Intelligence. He helps customers accelerate their AI adoption and build secure, scalable AI systems end-to-end, turning real-world edge cases into reusable patterns that help customers move faster. In his free time, Daniel enjoys playing soccer, running, and hiking.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-129438" height="129" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-8.png" width="100" />
  </div> 
  <p><strong>Ajit Kumar K.P.</strong> is a Senior Generative AI Partner Solutions Architect at AWS, where he works with enterprise customers and partners deploying AI solutions in the cloud. He brings deep expertise bridging the gap between platform engineering and enterprise-scale AI, having built Computer Vision solutions at the Edge, and AIML and Generative AI solutions in the Cloud. Ajit enjoys reading biographies and playing sports in his free time.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-full wp-image-129439" height="139" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/23/ML-20696-9-1.png" width="100" />
  </div> 
  <p><strong>Bharathan Balaji</strong>&nbsp;is a Senior Applied Scientist at Amazon Web Services, working on reinforcement learning and foundation model services. His work focuses on building AI capabilities that help customers transform their businesses.</p> 
 </div> 
</footer>
