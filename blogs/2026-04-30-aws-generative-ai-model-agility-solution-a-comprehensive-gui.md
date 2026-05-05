---
title: "AWS Generative AI Model Agility Solution: A comprehensive guide to migrating LLMs for generative AI production"
url: "https://aws.amazon.com/blogs/machine-learning/aws-generative-ai-model-agility-solution-a-comprehensive-guide-to-migrating-llms-for-generative-ai-production/"
date: "Thu, 30 Apr 2026 17:04:41 +0000"
author: "Long Chen"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p>Maintaining model agility is crucial for organizations to adapt to technological advancements and optimize their artificial intelligence (AI) solutions. Whether transitioning between different large language model (LLM) families or upgrading to newer versions within the same family, a structured migration approach and a standardized process are essential for facilitating continuous performance improvement while minimizing operational disruptions. However, developing such a solution is challenging in both&nbsp;technical and non-technical aspects because the solution needs to:</p> 
<ul> 
 <li>Be generic to cover a variety of use cases</li> 
 <li>Be specific so that a new user can apply it to the target use case</li> 
 <li>Provide comprehensive and fair comparison between LLMs</li> 
 <li>Be automated and scalable</li> 
 <li>Incorporate domain- and task-specific knowledge and inputs</li> 
 <li>Have a well-defined, end-to-end process from data preparation guidance to final success criteria</li> 
</ul> 
<p>In this post, we introduce a systematic framework for LLM migration or upgrade in generative AI production, encompassing essential tools, methodologies, and best practices. The framework facilitates transitions between different LLMs by providing robust protocols for prompt conversion and optimization. It includes evaluation mechanisms that assess multiple performance dimensions, enabling data-driven decision-making through detailed and comparative analysis of source and destination models. The proposed approach offers a comprehensive solution that includes the technical aspects of model migration and provides quantifiable metrics to validate successful migration and identify areas for further optimization, facilitating a seamless transition and continuous improvement. Here are a few highlights of the solution:</p> 
<ul> 
 <li>Provides a variety of reporting options with various LLM evaluation frameworks and comprehensive guidance for metrics selection for target use cases.</li> 
 <li>Provides automated prompt optimization and migration with <a href="https://aws.amazon.com/about-aws/whats-new/2024/11/prompt-optimization-preview-amazon-bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Prompt Optimization</a> and the <a href="https://anthropic.com/metaprompt-notebook" rel="noopener noreferrer" target="_blank">Anthropic Metaprompt tool</a>, in addition to best practices for further prompt optimization.</li> 
 <li>Provides comprehensive guidance for model selection and an end-to-end solution for model comparison regarding cost, latency, accuracy, and quality.</li> 
 <li>Provides feature examples and use case examples for users to quickly apply the solution to the target use case.</li> 
 <li>The total time required for an LLM migration or upgrade by following this framework is from two days up to two weeks depending on the complexity of the use case.</li> 
</ul> 
<h2>Solution overview</h2> 
<p><img alt="Process flow diagram showing LLM migration workflow from source models (OpenAI, Mistral, Llama, Claude) to Amazon Bedrock target models, including evaluation, comparison, and deployment phases." class="alignnone wp-image-128440 size-full" height="576" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-1.png" width="1224" /></p> 
<p>The core of the migration involves a three-step approach, shown in the preceding diagram.</p> 
<ol> 
 <li>Evaluate the source model.</li> 
 <li>Prompt migration to and&nbsp;optimization of the target model with <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-management-optimize.html" rel="noopener noreferrer" target="_blank">Amazon Bedrock prompt optimization</a> and the&nbsp;Anthropic Metaprompt tool.</li> 
 <li>Evaluate the target model.</li> 
</ol> 
<p>This solution provides a comprehensive approach to upgrade existing generative AI solutions (source model) to LLMs on Amazon Bedrock (target model). This solution addresses technical challenges through:</p> 
<ul> 
 <li>Evaluation metrics selection with a framework that uses various LLMs</li> 
 <li>Prompt improvement and migration with <a href="https://aws.amazon.com/about-aws/whats-new/2024/11/prompt-optimization-preview-amazon-bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Prompt Optimization</a> and&nbsp;the Anthropic Metaprompt tool</li> 
 <li>Model comparison across cost, latency, and performance</li> 
</ul> 
<p>This structured approach provides a robust framework for evaluating, migrating, and optimizing LLMs. By following these steps, we can transition between models, potentially unlocking improved performance, cost-efficiency, and capabilities in your AI applications. The process emphasizes thorough preparation, systematic evaluation, and continuous improvement; setting the stage for long-term success in using advanced language models.</p> 
<h2>Solution implementation</h2> 
<h3>Dataset preparation</h3> 
<p>An evaluation dataset with high-quality samples is critical to the migration process. For most use cases, samples with ground truth answers are required; while for other use cases, metrics that don’t require ground truth—such as&nbsp;answer relevancy,&nbsp;faithfulness, toxicity, and bias (see <em>Evaluation of frameworks and metrics selection</em> section)—can be used as the determination metrics.&nbsp;Use the following guidance and data format to prepare the sample data for the target use cases.</p> 
<p>Suggested fields for sample data include:</p> 
<ul> 
 <li>Prompt used for the source model</li> 
 <li>Prompt input (if any), for example: Questions and context for Retrieval-Augmented Generation (RAG)-based answer generation</li> 
 <li>Configurations used for&nbsp;source model invocation, for example, temperature, top_p, top_k, and so on.</li> 
 <li>Ground truths</li> 
 <li>Output from&nbsp;the source model</li> 
 <li>Latency of&nbsp;the source model</li> 
 <li>Input and output tokens from&nbsp;the source model, which can be used for cost calculation</li> 
</ul> 
<p>It’s important to remember that&nbsp;high quality ground truths are essential to successful migration for most use cases. Ground truths should not only be validated regarding correctness, but also to verify that they fit the subject matter expert’s (SME’s) guidance and evaluation criteria. See <em>Error </em><em>Analysis section</em>&nbsp;for an example of a SME’s guidance and evaluation criteria.</p> 
<p>In addition, if any existing evaluation metrics are available, such as a human evaluation score or thumbs up/thumbs down from a SME, include those metrics and corresponding reasoning or comments for each data sample. If any automated evaluations have been conducted, include the automated evaluation scores, methods, and configurations.&nbsp;The following section provides more detailed guidance on selecting evaluation frameworks and defining the metrics. However, it’s still valuable to collect the existing or preferred evaluation metrics from stakeholders for reference.</p> 
<p>Include the following fields if applicable:</p> 
<ul> 
 <li>Existing human evaluation metrics for the source model, for example, the SME score for source model.</li> 
 <li>Existing automated evaluation metrics for the source model, for example, the LLM-as-a-judge score for&nbsp;the source model.</li> 
</ul> 
<p>The following table is an example format of the data samples:</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>sample_id</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>…</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>question</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>content</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>prompt_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_ground_truth</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_ source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_ source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>input_token_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>output_token_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>llm_judge_score_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>human_score_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>human_score_reasoning_source_llm</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
 </tbody> 
</table> 
<h3>Evaluation of frameworks and metrics selection</h3> 
<p>After collecting information and data samples, the next step is to choose the proper evaluation metrics for the generative AI use case. Besides human evaluation by a SME, automated evaluation metrics are recommended because they are more scalable and objective and support the long-term health and sustainability of the product.&nbsp;The following table shows the automated metrics that are available for each use case.<img alt="" class="alignnone size-full wp-image-129249" height="922" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/20/image-26-2.png" width="1618" /></p> 
<h3>Model selection</h3> 
<p>The selection of an appropriate LLM requires careful consideration of multiple factors. Whether migrating to an LLM within the same LLM family or to a different LLM family, understanding the key characteristics of each model and the evaluation criteria is crucial for success.&nbsp;When planning to migrate between LLMs, carefully compare and evaluate various available options and check out the model card and respective prompting guides released by each model provider. When evaluating LLM options, consider several key criteria:</p> 
<ul> 
 <li><strong>Input and output modalities</strong>: Text, code, and multi-modal capabilities</li> 
 <li><strong>Context window size</strong>: Maximum input tokens the model can process</li> 
 <li><strong>Cost</strong> per inference or token</li> 
 <li><strong>Performance metrics</strong>: Latency and throughput</li> 
 <li><strong>Output</strong> quality and accuracy</li> 
 <li><strong>Domain specialization</strong> and specific use case compatibility</li> 
 <li><strong>Hosting options</strong>: Cloud, on-premises, and hybrid</li> 
 <li><strong>Data privacy and security</strong> requirements</li> 
</ul> 
<p>After initial filtering based on these characteristics, benchmarking tests should be conducted by evaluating performance on specific tasks to compare shortlisted models.&nbsp;Amazon Bedrock offers a comprehensive solution with access to various LLMs through a unified API. This allows us to experiment with different models, compare their performance, and even use multiple models in parallel, all while maintaining a single integration point.&nbsp;This approach not only simplifies the technical implementation but also helps avoid vendor lock-in by enabling a diversified AI model strategy.</p> 
<h3>Prompt migration</h3> 
<p>Two automated prompt migration and optimization tools are introduced here: the <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-management-optimize.html" rel="noopener noreferrer" target="_blank">Amazon Bedrock Prompt Optimization</a> and the&nbsp;Anthropic Metaprompt tool.</p> 
<h4><strong>Amazon Bedrock Prompt Optimization</strong></h4> 
<p><a href="https://aws.amazon.com/about-aws/whats-new/2024/11/prompt-optimization-preview-amazon-bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Prompt Optimization</a> is a tool available in Amazon Bedrock to automatically optimize prompts written by users. This helps&nbsp;users build high quality generative AI applications on Amazon Bedrock and reduces friction when moving workloads from other providers to Amazon Bedrock. Amazon Bedrock Prompt Optimization can enable migration of existing workloads from a source model to LLMs on Amazon Bedrock with minimal prompt engineering. With this tool,&nbsp;we can choose the model to optimize the prompt for and then generate an optimized prompt for the target model.&nbsp;The main advantage of using Amazon Bedrock Prompt Optimization is the ability to use it from the AWS Management Console for Amazon Bedrock. Using the console, we can quickly generate a new prompt for the target model. We can also use the <a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_OptimizePrompt.html" rel="noopener noreferrer" target="_blank">Bedrock API</a> to generate a migrated prompt, please see the detailed implementation below.</p> 
<h5><strong>Option A) Optimize a prompt from the Amazon Bedrock Console</strong></h5> 
<ol> 
 <li>In the Amazon Bedrock console, go to <strong>Prompt management</strong>.</li> 
 <li>Choose <strong>Create prompt</strong>, enter a name for the prompt template, and choose <strong>Create</strong>.</li> 
</ol> 
<p><img alt="Screenshot of AWS Create Prompt dialog box with name field showing &quot;openAI-to-Claude,&quot; optional description field, and KMS key encryption settings with Cancel and Create buttons" class="alignnone size-full wp-image-128444" height="486" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-2-1.jpeg" width="793" /></p> 
<ol start="3"> 
 <li>Enter the source model prompt. Create variables by enclosing a name with double curly braces: <code>{{variable}}</code>. In the <strong>Test variables</strong> section, enter values to replace the variables with when testing.</li> 
 <li>Select a <strong>Target Model for your optimized prompt</strong>. For example, Anthropic’s Claude Sonnet 4.</li> 
</ol> 
<p><img alt="AWS Bedrock Select Model dialog showing Anthropic provider selected with Claude model variants including Claude Sonnet 4, inference profile configuration, and cross-region deployment settings." class="alignnone size-full wp-image-128438" height="668" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-3-1.png" width="801" /></p> 
<ol start="5"> 
 <li>Choose the <strong>Optimize</strong> button to generate an optimized prompt for the target model.</li> 
</ol> 
<p><img alt="Prompt configuration interface showing instructions for creating variables using double curly brace syntax with an Optimize button for AI prompt improvement." class="alignnone size-full wp-image-128436" height="236" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-4-1.png" width="1594" /></p> 
<p>6. After the prompt is generated, the comparison window of the optimized prompt for the target model is shown with your original prompt from source model.</p> 
<p><img alt="Side-by-side prompt comparison interface showing a simple original prompt versus an optimized Variant_1 with structured XML-style tags, task definitions, and response format templates using Claude 3.5 Sonnet." class="alignnone size-full wp-image-128435" height="1340" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-5-1.png" width="3184" /></p> 
<p>7. Save the new optimized prompt before exiting comparing mode.</p> 
<h5><strong>Option B) Optimize a prompt using Amazon Bedrock API</strong></h5> 
<p>We can also use the Bedrock API to generate a migrated prompt, by sending an&nbsp;<a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_OptimizePrompt.html" rel="noopener noreferrer" target="_blank">OptimizePrompt&nbsp;</a>request with an&nbsp;<a href="https://docs.aws.amazon.com/general/latest/gr/bedrock.html#bra-rt" rel="noopener noreferrer" target="_blank">Agents for Amazon Bedrock runtime endpoint</a>. Provide the prompt to optimize in the&nbsp;input&nbsp;object and specify the model to optimize for in the&nbsp;<code>targetModelId</code>&nbsp;field.</p> 
<p>The response stream returns the following events:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_AnalyzePromptEvent.html" rel="noopener noreferrer" target="_blank">analyzePromptEvent</a>&nbsp;– Appears when the prompt is finished being analyzed. Contains a message describing the analysis of the prompt.</li> 
 <li><a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_OptimizedPromptEvent.html" rel="noopener noreferrer" target="_blank">optimizedPromptEvent</a>&nbsp;– Appears when the prompt has finished being rewritten. Contains the optimized prompt.</li> 
</ol> 
<p>Run the following code sample to optimize a prompt:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3

# Set values here
TARGET_MODEL_ID = "anthropic.claude-3-sonnet-20240229-v1:0" # Model to optimize for. For model IDs, see https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids.html
PROMPT = "Please summarize this text: " # Prompt to optimize

def get_input(prompt):
    return {
        "textPrompt": {
            "text": prompt
        }
    }
 
def handle_response_stream(response):
    try:
        event_stream = response['optimizedPrompt']
        for event in event_stream:
            if 'optimizedPromptEvent' in event:
                print("========================== OPTIMIZED PROMPT ======================\n")
                optimized_prompt = event['optimizedPromptEvent']
                print(optimized_prompt)
            else:
                print("========================= ANALYZE PROMPT =======================\n")
                analyze_prompt = event['analyzePromptEvent']
                print(analyze_prompt)
    except Exception as e:
        raise e
 
 
if __name__ == '__main__':
    client = boto3.client('bedrock-agent-runtime')
    try:
        response = client.optimize_prompt(
            input=get_input(PROMPT),
            targetModelId=TARGET_MODEL_ID
        )
        print("Request ID:", response.get("ResponseMetadata").get("RequestId"))
        print("========================== INPUT PROMPT ======================\n")
        print(PROMPT)
        handle_response_stream(response)
    except Exception as e:
        raise e</code></pre> 
</div> 
<h4><strong>Anthropic Metaprompt tool</strong></h4> 
<p>The Metaprompt is a prompt optimization tool offered by Anthropic where Claude is prompted to write prompt templates on the user’s behalf based on a topic or task. We can use it to instruct Claude on how to best construct a prompt to achieve a given objective consistently and accurately.</p> 
<p>The key steps are:</p> 
<ol> 
 <li>Specify the raw prompt template, explain the task, and specify the input variables and the expected output.</li> 
 <li>Run Metaprompt with a Claude LLM such as Claude-3-Sonnet by inputting the raw prompt from the source model.</li> 
 <li>The new prompt template is generated with an optimized set of instructions and format following Claude LLM’s&nbsp;best practices.</li> 
</ol> 
<p>Benefits of using metaprompts:</p> 
<ul> 
 <li>Prompts are much more detailed and comprehensive compared to human-created prompts</li> 
 <li>Helps increase the likelihood that best practices are followed for prompting the Anthropic models</li> 
 <li>Allows specifying that key details such preferred tone</li> 
 <li>Improves quality and consistency of the model’s outputs</li> 
</ul> 
<p>The Metaprompt tool is particularly useful for learning Claude’s preferred prompt style or as a method to generate multiple prompt versions for a given task, simplifying testing a variety of initial prompt variations for the target use case.</p> 
<p>To implement this process, follow the steps in the<a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/blob/main/use-case-examples/rag-investment-analysis-assistant/notebooks/03_Prompt_migration.ipynb" rel="noopener noreferrer" target="_blank"> Prompt Migration Jupyter Notebook</a>&nbsp; to migrate source model prompts to target model prompts. This notebook requires Claude-3-Sonnet to be enabled as the LLM in Amazon Bedrock using Model Access to generate the converted prompts.</p> 
<p>The following is one example of a source model prompt in a financial Q&amp;A use case:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">To answer the financial question, think step-by-step:
1. Carefully read the question and any provided context paragraphs related to yearly and quarterly document reports to find all relevant paragraphs. Prioritize context paragraphs with CSV tables.
2. If needed, analyze financial trends and quarter-over-quarter (Q/Q) performance over the detected time spans mentioned in the related time keywords. Calculate rates of change between quarters to identify growth or decline.
3. Perform any required calculations to get the final answer, such as sums or divisions. Show the math steps.
4. Provide a complete, correct answer based on the given information. If information is missing, state what is needed to answer the question fully.
5. Present numerical values in rounded format using easy-to-read units.
6. Do not preface the answer with "Based on the provided context" or anything similar. Just provide the answer directly.
7. Include the answer with relevant and exhaustive information across all contexts. Substantiate your answer with explanations grounded in the provided context. Conclude with a precise, concise, honest, and to-the-point answer.
8. Add the page source and number.
9. Add all source files from where the contexts were used to generate the answers.
context = {CONTEXT}
query = {QUERY}
rephrased_query = {REPHARSED_QUERY}
time_kwds = {TIME_KWDS}
</code></pre> 
</div> 
<p>After completing the steps in the notebook, we can automatically get the optimized prompt for the target model.&nbsp;The following example generates a prompt optimized for Anthropic’s Claude LLMs.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">Here are the steps to answer the financial question:

1. Read the provided &lt;context&gt;{$CONTEXT}&lt;/context&gt; carefully, paying close attention to any paragraphs and CSV tables related to yearly and quarterly financial reports. Prioritize context paragraphs containing CSV tables.

2. Identify the relevant time periods mentioned in the &lt;time_kwds&gt;{$TIME_KWDS}&lt;/time_kwds&gt;. Analyze the financial trends and quarter-over-quarter (Q/Q) performance during those time spans. Calculate rates of change between quarters to determine growth or decline.

3. &lt;scratchpad&gt;
In this space, you can perform any necessary calculations to arrive at the final answer to the &lt;query&gt;{$QUERY}&lt;/query&gt; or &lt;rephrasedquery&gt;{$REPHARSED_QUERY}&lt;/rephrasedquery&gt;. Show your step-by-step work, including formulas used and intermediate values.
&lt;/scratchpad&gt;

4. &lt;answer&gt;
Provide a complete and correct answer based on the information given in the context. If any crucial information is missing to fully answer the question, state what additional details are needed.

Present numerical values in an easy-to-understand format using appropriate units. Round numbers as necessary.

Do not include any preamble like "Based on the provided context..." Just provide the direct answer.

Include all relevant and exhaustive information from the contexts to substantiate your answer. Explain your reasoning grounded in the provided evidence. Conclude with a precise, concise, honest, and to-the-point final answer.

Finally, cite the page source and number, as well as list all files that contained context used to generate this answer.
&lt;/answer&gt;</code></pre> 
</div> 
<p>As shown in the preceding example, the prompt style and format are automatically converted to follow the best practices of the target model, such as using XML tags and regrouping the instructions to be clearer and more direct.</p> 
<h3>Generate results</h3> 
<p>Answer generation during migration is an iterative process. The general flow includes passing migrated prompts and context to the LLM and generating an answer. Multiple iterations are needed to compare different prompt versions, multiple LLMs, and different configurations of each LLM to help us select the best combination. In most cases, the entire pipeline of a generative AI system (such as a RAG-based chatbot) isn’t migrated. Instead, only a portion of the pipeline is migrated. Thus, it’s crucial that a fixed version of the remaining components in the pipeline is available. For example, in a RAG-based question and answer (Q&amp;A) system, we might migrate only the answer generation component of the pipeline. As a result, we can continue to use the already generated context of the existing production model.</p> 
<p>As a best practice, use the Amazon Bedrock models standard invocation method (in the <a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/tree/main" rel="noopener noreferrer" target="_blank">Migration code repository</a>) to generate metadata such as latency, time to first token, input token, and output token in addition to the final response. These metadata fields are added as a new column at the end of the results table and used for evaluation. The output format and column name should be aligned with the evaluation metric requirements. The following table shows an example of the sample data before feeding it into the evaluation pipeline for a RAG use case.</p> 
<p>Example of a sample data before evaluation:</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>financebench_id</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">financebench_id_03029</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_name</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">3M_2018_10K</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_link</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">https://investors.3m.com/financials/sec-filings/content/0001558370-19-000470/0001558370-19-000470.pdf</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_period</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2018</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>question_type</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">metrics-generated</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>question</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">What is the FY2018 capital expenditure amount (in USD millions) for 3M? Give a response to the question by relying on the details shown in the cash flow statement.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>ground_truths</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">[‘$1577.00’]</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>evidence_text</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">…</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>page_number</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">60</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>llm_answer</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">According to the cash flow statement in the 3M 2018 10-K report, the capital expenditure (purchases of property, plant and equipment) for fiscal year</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>llm_contexts</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">…</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_time</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.92706</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_kwd</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.60666</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_comb</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">1.44876</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_ans_gen</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2.48371</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>input_tokens</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">21147</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>output_tokens</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">401</td> 
  </tr> 
 </tbody> 
</table> 
<h3>Evaluation</h3> 
<p>Evaluation is one of the most important parts of the migration process because it directly connects to the sign-off criteria and determines the success of the migration. For most cases, evaluation focuses on metrics in three major categories: accuracy and quality, latency, and cost.&nbsp;Either automated evaluation or human evaluation can be used to assess the accuracy and quality of the model response.</p> 
<h4>Automated evaluation</h4> 
<p>The integration of LLMs in the quality evaluation process represents a significant advancement in assessment methodology. These models excel at conducting comprehensive evaluations across multiple dimensions, including contextual relevance, coherence, and factual accuracy, while maintaining consistency and scalability.&nbsp;Two&nbsp;primary categories of the automated evaluation metrics are introduced here:</p> 
<ol> 
 <li><strong>Predefined metrics</strong>: Metrics predefined in&nbsp;LLM-based evaluation frameworks such as <a href="https://docs.ragas.io/en/latest/" rel="noopener noreferrer" target="_blank">Ragas</a>, <a href="https://docs.confident-ai.com/" rel="noopener noreferrer" target="_blank">DeepEval</a>, and <a href="https://aws.amazon.com/bedrock/evaluations/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Evaluations</a>, or directly based on non-LLM algorithms, like those introduced in <a href="#_Evaluation_of_frameworks" rel="noopener noreferrer" target="_blank"><em>Evaluation of frameworks</em></a>.</li> 
 <li><strong>Custom metrics</strong>: Customized&nbsp;metrics with user provided definitions, evaluation criteria, or prompts to use LLM as an impartial judge.</li> 
</ol> 
<p><strong>Predefined metrics</strong></p> 
<p>These metrics are either using some LLM-based evaluation frameworks such as Ragas and DeepEval or are directly based on non-LLM algorithms. These metrics are widely adopted, predefined, and have limited options for customization. Ragas and DeepEval are two LLM-based evaluation frameworks and metrics that we used as examples in the&nbsp;<a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/tree/main" rel="noopener noreferrer" target="_blank">Migration code repository</a>.</p> 
<ul> 
 <li><strong>Ragas: </strong>Ragas is an open source framework that helps to evaluate RAG pipelines. RAG denotes a class of LLM applications that use external data to augment the LLM’s context.&nbsp;It provides a variety of LLM-powered automated evaluation metrics.&nbsp;The following metrics are introduced in the <a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/blob/main/use-case-examples/rag-investment-analysis-assistant/notebooks/05b_Evaluate_QnA_RAGAS.ipynb" rel="noopener noreferrer" target="_blank">Ragas evaluation notebook</a> in the Migration code repository. 
  <ul> 
   <li><strong>Answer precision:&nbsp;</strong>Measures how accurately the model’s generated answer contains relevant and correct claims compared to the ground truth answer.</li> 
   <li><strong>Answer recall:&nbsp;</strong>Evaluates the completeness of the answer; that is, the model’s ability to retrieve the&nbsp;correct claims and compare them to the ground truth answer. High recall indicates that the answer thoroughly covers the necessary details in line with the ground truth.</li> 
   <li><strong>Answer correctness:&nbsp;</strong>The assessment of answer correctness involves gauging the accuracy of the generated answer when compared to the ground truth. This evaluation relies on the <code>ground truth</code> and the <code>answer</code>, with scores ranging from 0 to 1. A higher score indicates a closer alignment between the generated answer and the ground truth, signifying better correctness.</li> 
   <li><strong>Answer similarity:&nbsp;</strong>The assessment of the semantic resemblance between the generated answer and the ground truth. This evaluation is based on the <code>ground truth</code> and the <code>answer</code>, with values falling within the range of 0 to 1. A higher score signifies a better alignment between the generated answer and the ground truth.</li> 
  </ul> </li> 
</ul> 
<p>The following table is a sample data output after Ragas evaluation.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>financebench_id</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">financebench_id_03029</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_name</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">3M_2018_10K</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_link</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://investors.3m.com/financials/sec-filings/content/0001558370-19-000470/0001558370-19-000470.pdf" rel="noopener noreferrer" target="_blank">https://investors.3m.com/financials/sec-filings/content/0001558370-19-000470/0001558370-19-000470.pdf</a></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>doc_period</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2018</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>question_type</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">metrics-generated</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>question</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">What is the FY2018 capital expenditure amount (in USD millions) for 3M?.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>ground_truths</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">[‘$1577.00’]</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>evidence_text</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">…</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>page_number</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">60</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>llm_answer</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">According to the cash flow statement in the 3M 2018 10-K report, the capital expenditure (purchases of property, plant and equipment) for fiscal year 2018 was $1,577 million. …</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>llm_contexts</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">…</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_time</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.92706</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_kwd</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.60666</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_comb</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">1.44876</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>latency_meta_ans_gen</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2.48371</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>input_tokens</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">21147</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>output_tokens</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">401</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_precision</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_recall</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">1</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_correctness</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.16818</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>answer_similarity</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">0.33635</td> 
  </tr> 
 </tbody> 
</table> 
<ul> 
 <li><strong>DeepEval: </strong>DeepEval is an open source LLM evaluation framework. It’s similar to Pytest but specialized for unit testing LLM outputs. DeepEval incorporates the latest research to evaluate LLM outputs based on metrics such as the G-Eval, hallucination, answer relevancy, Ragas, and so on. It uses LLMs and various other natural language processing (NLP) models that run locally on your machine for evaluation. In DeepEval, a metric serves as a standard of measurement for evaluating the performance of an LLM output based on specific criteria. DeepEval offers a range of default metrics to quickly get started. The following metrics are introduced in the <a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/blob/main/use-case-examples/rag-investment-analysis-assistant/notebooks/05a_Evaluate_QnA_DeepEval.ipynb" rel="noopener noreferrer" target="_blank">DeepEval evaluation notebook</a> in the Migration code repository.| 
  <ul> 
   <li><strong>Answer relevancy</strong>: The answer relevancy metric measures the quality of your RAG pipeline’s generator by evaluating how relevant the <code>actual_output</code> of your LLM application is compared to the provided input.</li> 
   <li><strong>Faithfulness</strong>: The faithfulness metric measures the quality of your RAG pipeline’s generator by evaluating whether the <code>actual_output</code> factually aligns with the contents of your <code>retrieval_context</code>.</li> 
   <li><strong>Toxicity</strong>: The toxicity metric is another referenceless metric that evaluates toxicity in your LLM outputs.</li> 
   <li><strong>Bias</strong>: The bias metric determines whether your LLM output contains gender, racial, or political bias.</li> 
  </ul> </li> 
</ul> 
<ul> 
 <li><strong>Amazon Bedrock Evaluations</strong>: Amazon Bedrock Evaluations is a suite of tools for evaluating, comparing, and selecting foundation models – including custom or third-party models – for your specific use cases. It supports both model-only and RAG pipelines evaluation. We can use Bedrock Evaluations either via AWS console or API. Amazon Bedrock Evaluations offers an extensive list of built-in metrics for both standalone LLMs and full RAG pipelines, including but not limited to: 
  <ul> 
   <li><strong>Accuracy</strong>: Measures the correctness of model outputs.</li> 
   <li><strong>Faithfulness: </strong>Checks for factual accuracy and avoids hallucinations.</li> 
   <li><strong>Helpfulness</strong>: Measures holistically how useful responses are in answering questions.</li> 
   <li><strong>Logical coherence</strong>: Measures whether the responses are free from logical gaps, inconsistencies or contradictions.</li> 
   <li><strong>Harmfulness</strong>: Measures harmful content in the responses, including hate, insults, violence, or sexual content.</li> 
   <li><strong>Stereotyping</strong>: Measures generalized statements about individuals or groups of people in responses.</li> 
   <li><strong>Refusal</strong>: Measures how evasive the responses are in answering questions.</li> 
   <li><strong>Following instructions</strong>: Measures how well the model’s response respects the exact directions found in the prompt.</li> 
   <li><strong>Professional style and tone</strong>: Measures how appropriate the response’s style, formatting, and tone is for a professional setting.</li> 
  </ul> </li> 
</ul> 
<p><strong>Custom metrics&nbsp;</strong></p> 
<p>These metrics are user defined and are typically tailored to specific tasks or domains. One popular method is to use custom LLM as a judge to provide an evaluation score for an answer using a user-provided prompt. In contrast to using predefined metrics, this method is highly&nbsp;customizable because we can provide the prompt with task-specific evaluation requirements. For example, we can&nbsp;ask the LLM to generate a 10-point scoring system and comprehensively evaluate the answer against ground truth across different dimensions, such as correctness of information, contextual relevance, depth and comprehensiveness of detail, and overall utility and helpfulness.</p> 
<p>The following is an example of a customized prompt for LLM as a judge:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">#Prompt: &nbsp; 
System: "You are an AI evaluator that helps in evaluating output from LLM",
&nbsp;
resp_fmt = """{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "score":float,
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; "reasoning": str
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp;&nbsp; &nbsp; &nbsp; """
&nbsp;
User = f"""[Instruction]\nPlease act as an impartial judge and evaluate the quality of the response
&nbsp;&nbsp; &nbsp;provided by an AI assistant to the user question displayed below. Your evaluation should consider correctness,
&nbsp;&nbsp; &nbsp;relevance, level of detail and helpfulness. You will be given a reference answer and the assistant's answer.
&nbsp;&nbsp; &nbsp;Begin your evaluation by comparing the assistant's answer with the reference answer. Identify any mistakes. Be as
&nbsp;&nbsp; &nbsp;objective as possible. After providing your explanation in the "reasoning" tab , you must score the response on a
&nbsp;&nbsp; &nbsp;scale of 1 to 10 in the "score" tab. Strictly follow the below json format:{resp_fmt}.
&nbsp;&nbsp; \n\n[Question]\n{question}\n\n[The Start of Reference Answer]\n{reference}\n[The End of Reference Answer]\n\n[The
&nbsp;&nbsp; &nbsp;Start of Assistant's Answer]\n{response}\n[The End of Assistant's Answer]"""</code></pre> 
</div> 
<h4>Human evaluation</h4> 
<p>While quantitative metrics provide valuable data points, a comprehensive qualitative evaluation based on professional guidelines and SME feedback is also necessary to validate model performance.&nbsp;Effective qualitative assessment typically covers several key areas including response theme and tone consistency, detection of inappropriate or unwanted content, domain-specific accuracy, date and time related issues, and so on.&nbsp;By using SME expertise, we can identify subtle nuances and potential issues that might escape quantitative analysis.&nbsp;<a href="#_Error_analysis" rel="noopener noreferrer" target="_blank">Error analysis</a> provides some potential aspects that the SME can use for evaluation criteria, which can also serve as the guidance for validating and preparing ground truths. We can use tools such as Amazon Bedrock Evaluations for human evaluation.</p> 
<p>Though human evaluation or user feedback collected from a UI can directly reflect the SME’s evaluation&nbsp;criteria, it’s not as efficient, scalable, and objective as the automated evaluation methods. Thus, a generative AI system development life cycle might start with human evaluation but eventually moves toward automated evaluation. Human&nbsp;evaluation can be used if automated&nbsp;evaluation isn’t meeting baseline targets or pre-defined evaluation criteria.</p> 
<h4>Latency metrics</h4> 
<p>When migrating language models, runtime performance metrics are crucial indicators of operational success. Total latency and Time to first token (TTFT) are the most common metrics for latency measurement.</p> 
<ul> 
 <li><strong>Total latency</strong> is an end-to-end metric that measures the total duration required for complete response generation, from initial prompt to final output.&nbsp;It encompasses processing the input, generating the response, and delivering it to the user. Total latency affects user satisfaction, system throughput, and resource utilization.</li> 
 <li><strong>Time to first token (TTFT)</strong> quantifies the initial response speed—specifically, the duration until the model generates its first output token. This metric significantly impacts perceived responsiveness and user experience, especially in interactive applications.&nbsp;TTFT is particularly important in conversational AI and real-time systems (applications such as chatbots, virtual assistants, and interactive search systems) where users expect immediate feedback. A low TTFT creates an impression of system responsiveness and can greatly enhance user engagement.</li> 
</ul> 
<p>If the results generation step requires multiple LLM calls, the breakdown latency metrics should be provided because only the submodule latency corresponding to LLM migration should be compared in the following model comparison step.</p> 
<h4>Cost calculation</h4> 
<p>For LLM invocation, the cost can be calculated based on the number of input and output tokens and the corresponding price per token:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">LLM_invocation_cost = number_of_input_tokens * price_per_input_token +&nbsp;number_of_output_tokens * price_per_output_token</code></pre> 
</div> 
<p>The cost calculations table for price per input and output token can be found in <a href="https://aws.amazon.com/bedrock/pricing/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Pricing </a>.</p> 
<h3>Model comparison report: Performance, latency, and cost</h3> 
<p>We can use the <a href="https://github.com/aws-samples/Prompt-Migration-OpenAI-to-Amazon-Bedrock/blob/main/use-case-examples/rag-investment-analysis-assistant/notebooks/06_Generate_Comparision_Report.ipynb" rel="noopener noreferrer" target="_blank">Generate Comparison Report&nbsp;notebook</a> in the code repository to automatically generate a final comparison report for the source and target model in a holistic view.</p> 
<p>We can also use evaluation reports generated from Ragas and DeepEval with corresponding metrics to compare the models from the two evaluation frameworks. We can obtain a side-by-side comparison of the average input and output tokens and average cost and latency for the selected models.&nbsp;As shown in the following figure, after running this notebook, there are two comparison tables for the source and target models from the two selected evaluation frameworks.</p> 
<p><strong>Ragas<img alt="Performance comparison table showing GPT-4 Turbo versus Claude 3 Haiku metrics including precision, recall, correctness, similarity, token usage, cost, and latency." class="alignnone size-full wp-image-128434" height="206" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-6-scaled.jpeg" width="2560" /></strong></p> 
<p><strong>DeepEval<img alt="Model performance metrics table comparing GPT-4 Turbo and Claude 3 Haiku on answer relevance, faithfulness, bias, toxicity, token usage, cost, and latency." class="alignnone size-full wp-image-128441" height="187" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-7-scaled.jpeg" width="2560" /></strong></p> 
<h2>Further optimization</h2> 
<p>When enhancing and optimizing a generative AI production pipeline during an LLM migration or upgrade, users typically focus on two key areas:</p> 
<ol> 
 <li>Quality of generated answers</li> 
 <li>Latency of response generation</li> 
</ol> 
<h3>Prompt optimization</h3> 
<p>To optimize the quality of the generated answers, we need to get a good understanding of the errors by conducting error analysis and identifying the items for prompt optimization.</p> 
<h4>Error analysis</h4> 
<p>Getting the best possible response from a candidate LLM is unlikely without any optimization. Thus, conducting error analysis and focusing on possible aspects for error patterns helps us evaluate generated answer quality and identify the opportunities for improvement. Error analysis also provides a path to manual prompt engineering to improve the quality.&nbsp;After gathering error analysis insights and feedback from SMEs, an iterative prompt optimization process can be conducted. To start, formulate the&nbsp;error analysis insights and feedback from SMEs into clear guidance or criteria.&nbsp;Ideally, these&nbsp;criteria should be clarified before starting the prompt migration. These&nbsp;criteria serve as the core considerations for further prompt optimization&nbsp;to help provide consistent, high-quality responses to meet the SME’s bar.&nbsp;The following is an example of possible&nbsp;guidance and criteria we might receive from a SME.</p> 
<p><strong>Example of an answer formatting style guide from a SME in a financial Q&amp;A use case:<br /> </strong></p> 
<ul> 
 <li><strong>Correctness</strong> 
  <ul> 
   <li>Make sure pulled numbers are correct. All numbers should be matched to ground truth.</li> 
   <li>Make sure all claims from ground truth are available in the LLM answer.</li> 
   <li>Generated responses should not add irrelevant sentences.</li> 
  </ul> </li> 
 <li><strong>Time</strong> 
  <ul> 
   <li>Generated answers must recognize the fiscal year and all needed quarters from the question correctly.</li> 
   <li>In the answer,&nbsp;quarter orders from most recent to the earliest is preferred.</li> 
   <li>When the question asks about year-over-year, the answer should specify overall year or the last quarter, not quarter-by-quarter.</li> 
   <li>When the answer comes from a single news document, include the date of publication in the answer.</li> 
  </ul> </li> 
 <li><strong>Theme and tone</strong> 
  <ul> 
   <li>Use professional language mirroring the style of a newspaper.</li> 
  </ul> </li> 
 <li><strong>Format and excerpts</strong> 
  <ul> 
   <li>When the user query asks for a list, present the list in bullet point format.</li> 
   <li>When the user query asks for excerpts, provide a summary statement followed by a bulleted list of unedited excerpts directly from the document.</li> 
   <li>Queries that ask for a comprehensive list ideally include bullet points.</li> 
   <li>Queries that ask for topics or themes with subjective categories ideally include a bulleted list.</li> 
   <li>Don’t start the answer by referencing the context (according to context).</li> 
  </ul> </li> 
 <li><strong>Length</strong> 
  <ul> 
   <li>Most responses should be between 30–150 words. Longer answers are acceptable when the question involves multiple entities or responding to queries that require sub-categories within the response.</li> 
  </ul> </li> 
</ul> 
<h4>Optimization techniques</h4> 
<p>After obtaining clear&nbsp;criteria, several optimization techniques can be used to address these criteria, such as:</p> 
<ul> 
 <li>Prompt engineering to specify certain&nbsp;criteria in the instruction of the prompt</li> 
 <li>Few-shot learning to specify the answer format and generated answer examples</li> 
 <li>Incorporating meta-information that could help the LLM to understand the context of the task and question</li> 
 <li>Pre- or post-processing to enforce the output format or resolve some frequent error patterns</li> 
</ul> 
<h3>Latency optimization</h3> 
<p>There are a few possible solutions to optimize the latency:</p> 
<h4>Optimizing prompts to generate shorter answers</h4> 
<p>The latency of an LLM model is directly impacted by the number of output tokens because each additional token requires a separate forward pass through the model, increasing processing time. As more tokens are generated, latency grows, especially in larger models such as Opus 4. To reduce the latency, we can add instructions to prompt to avoid providing lengthy answers, unrelated explanations, or filler words.</p> 
<h4>Using&nbsp;provisioned throughput</h4> 
<p>Throughput refers to the number and rate of inputs and outputs that a model processes and returns. Purchasing&nbsp;provisioned throughput to provide a higher level of throughput for a dedicated hosted model can potentially reduce the latency compared to using on-demand models. Though it cannot guarantee the improvement of latency, it consistently helps to prevent throttled requests.</p> 
<h4>Improvement lifecycle</h4> 
<p>It’s unlikely that a candidate LLM can achieve the best possible performance without any optimization. It’s also typical for the preceding optimization processes to be conducted iteratively. Thus, the improvement (optimization) lifecycle is critical to improve the performance and identify the gaps or defects in the pipeline or data. The improvement lifecycle typically includes:</p> 
<ul> 
 <li>Prompt optimization</li> 
 <li>Answer generation</li> 
 <li>Evaluation metrics generation</li> 
 <li>Error analysis</li> 
 <li>Sample label verification</li> 
 <li>Dataset updates regarding sample defects and wrong labels</li> 
</ul> 
<p>Task or domain knowledge identificationThe migration process described in this post can be used in two phases in a generative AI solution production lifecycle.</p> 
<h3>End-to-end LLM migration and model agility</h3> 
<p>New LLMs are released frequently. No LLM can consistently maintain peak performance for a given use case. It’s common for a production generative AI solution to migrate to another family of LLMs or upgrade to a new version of an LLM. Thus, having a standard and reusable end-to-end LLM migration or upgrade process is critical to the long-term success of any generative AI solution.</p> 
<h3>Monitoring and quality assurance</h3> 
<p>When migration or updates are stabilized, there should be a standard&nbsp;monitoring and quality assurance process using a routinely refreshed golden evaluation dataset with ground truth and automated or human evaluation metrics, as well as evaluation of actual user traces. As part of this solution, the established evaluation and data or ground truth collection&nbsp;processes can be reused for monitoring and quality assurance.</p> 
<h2>Tips and suggestions (lessons learned)</h2> 
<p>The following are some tips and suggestions for the success of an LLM migration or upgrade process.</p> 
<ul> 
 <li><strong>Sign-off condition</strong>:<strong>&nbsp;</strong>The data, evaluation criteria and success criteria defined at the start should be&nbsp;sufficient for stakeholders to confidently sign off on the process. Ideally, there should be no changes in the data, ground truths, or SME evaluation and success criteria during the process.</li> 
 <li><strong>Sample data and quality</strong>: The data should be of sufficient quality and quantity for confident evaluation. The ground truth answers and labels should be fully aligned with the SME’s evaluation&nbsp;criteria and expectations. Ideally, there should be no changes in the data, ground truths, or SME evaluation criteria during the process.</li> 
 <li><strong>Improvement lifecycle</strong>: Make sure to plan and implement an improvement lifecycle to get the most out of your chosen LLM.</li> 
 <li><strong>Model selection</strong>: When selecting competing target models against a source model, use resources such as the <a href="https://artificialanalysis.ai/" rel="noopener noreferrer" target="_blank">Artificial Analysis</a>&nbsp;benchmarking website to obtain a holistic comparison of models. These comparisons typically cover quality, performance, and price analysis, providing valuable insights before starting the experiment. This preliminary research can help narrow down the most promising candidates and inform the experimental design.</li> 
 <li><strong>Performance against cost trade-offs</strong>: When evaluating different models or solutions, it’s important to consider the balance between performance and cost. In some cases, a model might offer slightly lower performance but at a sufficiently reduced cost to make it a more cost-effective option overall. This is particularly true in scenarios where the performance difference is minimal, but the cost savings are substantial.</li> 
 <li><strong>Optimization techniques</strong>: Exploring various optimization techniques, such as prompt engineering or provisioned throughput, can lead to significant improvements in performance metrics like accuracy and latency. These optimizations can help bridge the gap between different models and should be considered as part of the evaluation process.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post, we introduced the AWS Generative AI Model Agility Solution, an end-to-end solution for LLM migrations and upgrades of existing generative AI applications that maintains and improves model agility. The solution defines a standardized process and provides a comprehensive toolkit for LLM migration or upgrade with a variety of ready-to-use tools and advanced techniques that can can be used to migrate generative AI applications to new LLMs.&nbsp;This can be used as a standard process in the lifecycle of your&nbsp;generative AI applications. After an application is stabilized with a specific LLM and configuration, the evaluation and data and ground truth collection&nbsp;processes in this solution can be reused for production monitoring and quality assurance.</p> 
<p>To learn more about this solution, please check out our <a href="https://github.com/aws-samples/prompt-migration-for-large-language-model-agility/tree/main" rel="noopener noreferrer" target="_blank">AWS Generative AI Model Agility Code Repo</a>.</p> 
<hr /> 
<h2><strong>About the authors</strong></h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a man wearing black-framed rectangular glasses and a black polo shirt against a white background." class="wp-image-128442 size-thumbnail alignnone" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-8-100x100.jpeg" width="100" />
  </div> 
  <p><strong>Long Chen </strong>is a Sr. Applied Scientist at AWS Generative AI Innovation Center. He holds a Ph.D. in Applied Physics from University of Michigan – Ann Arbor. With more than a decade of experience for research and development, he works on innovative solutions in various domains using generative AI and other machine learning techniques, ensuring the success of AWS customers. His interests include generative models, multi-modal systems and graph learning.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-128447 size-full alignnone" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/elaine.png" width="100" />
  </div> 
  <p><strong>Elaine Wu </strong>is a Deep Learning Architect at the AWS Generative AI Innovation Center, specializing in building robust RAG and agentic AI solutions for large enterprises. She has solved real-world business challenges for AWS customers across industries including manufacturing, energy, healthcare, retail, enterprise software, and financial services. Prior to joining AWS, Elaine earned her master’s degree in Information Science from the University of Illinois Urbana-Champaign.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Portrait of a smiling person wearing round glasses and a pink knitted sweater, posing outdoors by a waterfront with a scenic lake background." class="alignnone wp-image-128437 size-thumbnail" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-10-100x100.png" width="100" />
  </div> 
  <p><strong>Samaneh Aminikhanghahi</strong> is an Applied Scientist at the AWS Generative AI Innovation Center, where she works with customers across different verticals to accelerate their adoption of generative AI. She specializes in agentic AI frameworks, building robust evaluation systems, and implementing responsible AI practices that drive sustainable business outcomes.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-128448 size-full" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/avinash.png" width="100" />
  </div> 
  <p><strong>Avinash Yadav</strong> is a Deep Learning Architect at the Generative AI Innovation Center, where he designs and implements cutting-edge GenAI solutions for diverse enterprise needs. He specializes in building agentic AI systems and multi-agent frameworks, developing AI agents capable of complex reasoning, tool use, and orchestration across enterprise workflows. His expertise spans ML pipelines using large language models, agentic architectures leveraging frameworks such as LangGraph and Amazon Bedrock AgentCore, along with cloud architecture, Infrastructure as Code (IaC), and automation. His focus lies in creating scalable, end-to-end applications that harness the power of deep learning, agentic workflows, and cloud technologies to solve real-world business challenges.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a man with dark hair wearing a light gray jacket over a maroon shirt against a dark background." class="alignnone wp-image-128433 size-thumbnail" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/13/ML-17637-image-12-100x100.jpeg" width="100" />
  </div> 
  <p><strong>Vidya Sagar Ravipati</strong>&nbsp;is a Science Manager at the Generative AI Innovation Center, where he leverages his vast experience in large-scale distributed systems and his passion for machine learning to help AWS customers across different industry verticals accelerate their AI and cloud adoption.</p> 
 </div> 
</footer>
