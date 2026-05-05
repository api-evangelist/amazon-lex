---
title: "Sun Finance automates ID extraction and fraud detection with generative AI on AWS"
url: "https://aws.amazon.com/blogs/machine-learning/sun-finance-automates-id-extraction-and-fraud-detection-with-generative-ai-on-aws/"
date: "Thu, 30 Apr 2026 17:00:45 +0000"
author: "Babs Khalidson"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p><em>This post was co-authored with Krišjānis Kočāns, Kaspars Magaznieks, Sergei Kiriasov from Sun Finance Group</em></p> 
<p>If you process identity documents at scale—loan applications, account openings, compliance checks—you’ve likely hit the same wall: traditional optical character recognition (OCR) gets you partway there, but extraction errors still push a large share of applications into manual review queues. Add fraud detection to the mix, and the manual workload compounds.</p> 
<p>Sun Finance, a Latvian fintech founded in 2017, operates as a technology-first online lending marketplace across nine countries. The company processes a new loan request every 0.63 seconds and delivers more than 4 million evaluations monthly. In one of their highest-volume industries, with 80,000 monthly applications for microloans, approximately 60% of applications required manual operator review. Sun Finance partnered with the <a href="https://aws.amazon.com/ai/generative-ai/innovation-center/" rel="noopener noreferrer" target="_blank">AWS Generative AI Innovation Center</a> to rebuild the pipeline. Within 35 business days of handover, the solution was live in production. The following timeline shows the full project journey from kickoff to production launch.</p> 
<p><img alt=" Project timeline spanning August 2025 to January 2026 showing key milestones: Kickoff (26 Aug 2025), Final Presentation (09 Oct 2025), Technical Handover (14 Nov 2025), and Live in Production (22 Jan 2026), with production freeze period from 18 Dec to 07 Jan" class="alignnone size-full wp-image-128926" height="669" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203161.png" width="1172" /></p> 
<p><em>Sun Finance project timeline from kickoff to production</em></p> 
<p>The project moved through four milestones over 107 business days. The AWS Generative AI Innovation Center engagement ran 32 days from kickoff (August 26, 2025) to final presentation (October 9, 2025), followed by 26 days for technical handover (November 14, 2025). Sun Finance then took 35 business days to move the solution into production, including a 14-day production freeze over the holiday period (December 18 – January 7), and went live on January 22, 2026.</p> 
<p>In this post, we show how Sun Finance used <a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a>, <a href="https://aws.amazon.com/textract/" rel="noopener noreferrer" target="_blank">Amazon Textract</a>, and <a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a> to build an AI-powered identity verification (IDV) pipeline. The solution improved extraction accuracy from 79.7% to 90.8%, cut per-document costs by 91%, and reduced processing time from up to 20 hours to under 5 seconds. You’ll learn how combining specialized OCR with large language model (LLM) structuring outperformed using either tool alone. You’ll also learn how to architect a serverless fraud detection system using vector similarity search.</p> 
<h2>The Identity Verification Challenge</h2> 
<p>Sun Finance had built its first IDV automation in 2019 using <a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a> and <a href="https://aws.amazon.com/textract/" rel="noopener noreferrer" target="_blank">Amazon Textract</a>. As the company expanded into developing regions, the system’s limitations became hard to ignore.</p> 
<p>This region presented unique challenges with language and document complexity. Processing documents in both English and a local language proved difficult for traditional OCR systems. The local language text remains underrepresented in traditional OCR training datasets, causing frequent extraction errors. Sun Finance also needed to handle 7 different ID types, each with different layouts and formats.</p> 
<p>The manual workload was primarily driven by OCR errors. Of the 60% of applications requiring manual review, approximately 80% of cases stemmed from mismatches between extracted information and customer-entered data. Critically, 60% of these mismatches were OCR errors, not customer mistakes. The remaining 20% of manual interventions related to fraud detection flags.</p> 
<p>Fraud detection added another layer of complexity. About 10% of daily requests were actual fraudulent applications. Fraudsters used similar images with distinctive patterns to bypass basic controls while submitting multiple loan applications. Identifying these patterns required time-intensive manual review across numerous images.</p> 
<p>Cost and speed constraints blocked expansion. The per-document cost and approximately 3 full-time equivalents (FTEs) dedicated to manual verification in this region alone meant the unit economics blocked expansion into industries with lower-value microloans. Processing times ranged from under 10 minutes for automated cases to 20 hours for manual reviews outside business hours.</p> 
<h2>Solution overview</h2> 
<p>The AWS Generative AI Innovation Center ran a 6-week proof-of-concept (September–October 2025) focused on one high-volume industry. The team built two AI-powered solutions: an ID extraction system and a fraud detection system. Both were deployed as a fully serverless architecture on AWS.The solution uses the following key services:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a> – For AI structuring and visual analysis using Anthropic’s Claude Sonnet 4, and vector generation using Amazon Titan Multimodal Embeddings.</li> 
 <li><a href="https://aws.amazon.com/textract/" rel="noopener noreferrer" target="_blank">Amazon Textract</a> – For primary OCR text extraction from identity documents.</li> 
 <li><a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a> – For fallback OCR, face detection, and face masking.</li> 
 <li><a href="https://aws.amazon.com/s3/vectors/" rel="noopener noreferrer" target="_blank">Amazon S3 Vectors</a> – For serverless vector similarity search against known fraud patterns.</li> 
 <li><a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a> – For orchestrating parallel fraud detection workflows.</li> 
 <li><a href="https://aws.amazon.com/lambda/" rel="noopener noreferrer" target="_blank">AWS Lambda</a> – For serverless compute across both pipelines.</li> 
</ul> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><img alt="AWS architecture diagram showing fraud detection and document processing pipeline using AWS Step Functions, Lambda functions, Amazon Rekognition, Amazon Textract, and Amazon Bedrock for automated image and document analysis" class="alignnone size-full wp-image-128927" height="3492" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203162.png" width="4412" /></p> 
<p><em>Sun Finance API architecture showing ID extraction and fraud detection routes</em></p> 
<p>The architecture exposes two API routes through Amazon API Gateway, with loan application data stored in Amazon Simple Storage Service (Amazon S3):</p> 
<ol> 
 <li><strong>`/extract-id` route (ID extraction).</strong> An AWS Lambda function receives the ID image and sends it to Amazon Textract for primary OCR. If Amazon Textract returns low-confidence results, the system falls back to Amazon Rekognition for OCR. The extracted text is then passed to Amazon Bedrock (Claude Sonnet 4), which structures it into standardized JSON fields.</li> 
 <li><strong>`/detect-fraud` route (fraud detection).</strong> An AWS Lambda function triggers an AWS Step Functions workflow that runs two checks in parallel: 
  <ul> 
   <li><strong>Background similarity</strong> — Amazon Rekognition masks the face from the selfie image, then Amazon Bedrock Titan Multimodal Embeddings generates a vector representation of the background. This vector is queried against Amazon S3 Vectors to find matches with known fraud patterns.</li> 
   <li><strong>Visual pattern detection</strong> — Amazon Bedrock (Claude Sonnet 4) analyzes the image for screen photo artifacts and digital manipulation.</li> 
  </ul> </li> 
</ol> 
<p>Both results feed into a Lambda-based risk assessment function that produces a combined fraud score as JSON.</p> 
<ol start="3"> 
 <li><strong>Fraud ingestion pipeline (right side).</strong> Confirmed fraud images are ingested from Amazon S3 through a Lambda function. The images are processed by Amazon Rekognition for face masking, vectorized by Amazon Bedrock Titan Embeddings, and stored in Amazon S3 Vectors. This grows the reference database over time.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To implement a similar solution, you need the following:</p> 
<ul> 
 <li>An <a href="https://aws.amazon.com/free/" rel="noopener noreferrer" target="_blank">AWS account</a> with permissions to create and manage <a href="https://aws.amazon.com/lambda/" rel="noopener noreferrer" target="_blank">AWS Lambda</a>, <a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a>, <a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a>, <a href="https://aws.amazon.com/textract/" rel="noopener noreferrer" target="_blank">Amazon Textract</a>, <a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a>, and <a href="https://aws.amazon.com/s3/vectors/" rel="noopener noreferrer" target="_blank">Amazon S3 Vectors</a> resources.</li> 
 <li><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener noreferrer" target="_blank">Amazon Bedrock model access</a> enabled for Anthropic Claude Sonnet 4 and Amazon Titan Multimodal Embeddings in your AWS Region.</li> 
 <li><a href="https://www.terraform.io/" rel="noopener noreferrer" target="_blank">Terraform</a> installed for infrastructure deployment.</li> 
 <li>Familiarity with Python and serverless architectures.</li> 
 <li>A dataset of identity document images for testing and validation.</li> 
</ul> 
<h2>Solution walkthrough</h2> 
<p>This section walks through the two core pipelines: ID extraction and fraud detection.</p> 
<h3>ID extraction pipeline</h3> 
<p>The ID extraction system didn’t arrive at its final design on day one. The team iterated through three distinct approaches over four weeks, and each failure pointed toward the next improvement. The following diagram shows how the pipeline evolved from a single Claude Sonnet 4 via Amazon Bedrock approach at 61.8% accuracy to the final multi-tier design at 90.8%.</p> 
<p><img alt="Comparative visualization of three ID extraction approaches showing progression from 61.8% efficiency (Claude Vision only) to 85.0% efficiency (with Amazon Textract) to 90.8% efficiency (with validation and Amazon Rekognition fallback)" class="alignnone size-full wp-image-128928" height="1144" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203163.png" width="2038" /></p> 
<p><em>ID extraction: evolution of approaches showing three iterations from 61.8% to 90.8% accuracy</em></p> 
<p><strong>Approach 1: Claude Sonnet 4 alone (61.8% accuracy).</strong> The team’s first attempt sent ID images directly to Anthropic’s Claude Sonnet 4 via <a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a> and asked it to extract fields as JSON. The results were disappointing: 61.8% overall accuracy, with ID number extraction at only 43%. The core issue was the model’s built-in safety protocols for handling personally identifiable information (PII). Claude is trained to limit processing of sensitive PII found on identity documents like driver’s licenses, passports, and national IDs. When presented with real ID images, the model triggered these privacy safeguards and refused to extract information from some files, which directly impacted performance. Additionally, even when extraction succeeded, certain fields (like ID numbers) showed poor accuracy because the model prioritized safety over precise character recognition on sensitive documents.</p> 
<p>The takeaway: while Claude excels at general document analysis and OCR tasks, its built-in privacy protections make it unsuitable for direct extraction from identity documents containing PII.</p> 
<p><strong>Approach 2: Amazon Textract + Claude structuring (85% accuracy).</strong> The breakthrough came when the team separated OCR from structuring. <a href="https://aws.amazon.com/textract/" rel="noopener noreferrer" target="_blank">Amazon Textract</a> handled raw text extraction from ID images. Claude Sonnet 4 then structured the output into 7 standardized fields: document type, date of birth, name, surname, middle name, ID number, and expiry date. This single change produced an 11.6% accuracy jump.</p> 
<p>This approach worked because Amazon Textract, as a specialized OCR service, doesn’t have the same PII refusal mechanisms as Claude, so it reliably extracted text from every ID image without triggering safety protocols. Once the text was extracted, Claude could focus on what it does best: intelligent structuring. Claude excelled at handling local language text with diacritical marks, inferring missing information from context, and applying document-specific extraction rules. These are tasks that traditional OCR alone couldn’t handle. By working with already-extracted text rather than raw ID images, Claude avoided its safety constraints.</p> 
<p>The takeaway: separating concerns allowed each tool to operate within its design parameters: Amazon Textract for reliable OCR and Claude for intelligent structuring.</p> 
<p><strong>Approach 3: Multi-tier OCR + validation (90.8% accuracy).</strong> The final iteration added <a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a> as a fallback for images where Amazon Textract struggled (typically low-quality scans, unusual document angles, or damaged IDs) plus validation rules for ID number formatting, date standardization, and document type normalization.</p> 
<p>The multi-tier architecture works as follows. Amazon Textract handles primary OCR. Amazon Rekognition provides backup extraction when Amazon Textract confidence is low. Claude structures the combined output, and validation rules catch formatting errors that slip through. ID numbers get padded to the correct length based on document type, and dates are standardized to YYYY-MM-DD format. These validation rules proved critical. They caught edge cases where OCR extracted correct characters but in inconsistent formats.</p> 
<p>The following chart shows the weekly accuracy progression across 585 test images. The team didn’t beat the baseline until Week 4, when they added Amazon Textract. Each iteration revealed new failure modes that informed the next architectural improvement.</p> 
<p><img alt="Line graph showing ID extraction accuracy improvement over 4 weeks from 69.8% baseline (Claude Vision only) to 90.8% final accuracy, with milestones at Week 3 (73.4% after prompt tuning), Week 4 (85.0% after adding Textract), and Week 5 (90.8% with recognition fallback and validation)" class="alignnone size-full wp-image-128929" height="1168" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203164.png" width="2062" /></p> 
<p><em>ID extraction: the journey to 90.8% accuracy showing weekly progress</em></p> 
<p>The takeaway: combining specialized OCR tools (Amazon Textract + Amazon Rekognition) with LLM structuring (Claude) and validation rules beats using a single tool alone for document extraction.</p> 
<h3>Fraud detection pipeline</h3> 
<p>The fraud detection system uses <a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a> to run two detection methods in parallel, then combines their scores into a final risk assessment.</p> 
<p><strong>Visual pattern detection.</strong> Claude Sonnet 4 via <a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a> analyzes submitted selfie images for signs of fraud: screen photos (visible bezels, scan lines, moiré patterns), screen glare and reflections, and digital manipulation artifacts. Images scoring 85% confidence or higher are flagged. The system ignores normal characteristics like blur, compression artifacts, and standard cropping to reduce false positives. Screen photo detection works well, with 95%+ confidence on known patterns.</p> 
<p><strong>Background similarity analysis.</strong> This component catches fraud rings, which are groups of fraudsters submitting selfies from the same location. The pipeline works in three steps. First, <a href="https://aws.amazon.com/rekognition/" rel="noopener noreferrer" target="_blank">Amazon Rekognition</a> masks faces to focus on the background. Then, <a href="https://aws.amazon.com/bedrock/titan/" rel="noopener noreferrer" target="_blank">Amazon Titan</a> Multimodal Embeddings generates a 1024-dimensional vector of the background. Finally, <a href="https://aws.amazon.com/s3/vectors/" rel="noopener noreferrer" target="_blank">Amazon S3 Vectors</a> searches for matches against known fraud patterns.</p> 
<p>The team tested both text-based and visual embeddings for similarity search. Text embeddings (having Claude describe the background, then comparing descriptions) achieved 91% accuracy but only 27.8% precision and 21.7% recall. Visual embeddings performed far better: 96% accuracy, 80% precision, and 52% recall.</p> 
<p><img alt="Technical comparison of text embeddings versus visual embeddings for FAISS-based similarity search, showing visual embeddings achieving 96.0% accuracy, 80.0% precision, 52.2% recall, and 63.2% F1-score compared to text embeddings' 91.0% accuracy, 27.8% precision, 21.7% recall, and 24.4% F1-score" class="alignnone size-full wp-image-128930" height="1806" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203165.png" width="3290" /></p> 
<p><em>Background similarity: visual features approach showing the pipeline and text vs visual embedding comparison</em></p> 
<p><strong>Risk assessment.</strong> The scoring algorithm weighs visual pattern detection (50%) and background similarity (50%) equally. Scores of 75+ indicate high-confidence fraud, 38–74 indicate medium confidence, and below 38 is classified as legitimate. The parallel execution architecture processes images in 3–5 seconds, down from 6–8 seconds when run sequentially.</p> 
<h3>Serverless architecture</h3> 
<p>The entire solution runs on <a href="https://aws.amazon.com/lambda/" rel="noopener noreferrer" target="_blank">AWS Lambda</a>, <a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a>, and <a href="https://aws.amazon.com/api-gateway/" rel="noopener noreferrer" target="_blank">Amazon API Gateway</a>. This design lets the team modify individual Lambda functions, test changes immediately, and deploy updates without downtime. This was critical during a 6-week engagement where the approach changed weekly.</p> 
<p>Authentication uses <a href="https://aws.amazon.com/cognito/" rel="noopener noreferrer" target="_blank">Amazon Cognito</a> with AWS SigV4 request signing. <a href="https://aws.amazon.com/waf/" rel="noopener noreferrer" target="_blank">AWS WAF</a> protects against common web security issues. Data is encrypted at rest with <a href="https://aws.amazon.com/kms/" rel="noopener noreferrer" target="_blank">AWS Key Management Service (AWS KMS)</a> and in transit via TLS 1.2+. The infrastructure is defined in Terraform and passed security audits with 25 findings analyzed: 14 false positives, 9 justified exceptions, and 2 deferred for production.</p> 
<h2>Results</h2> 
<p>The proof-of-concept delivered measurable improvements across accuracy, speed, fraud detection, and cost.</p> 
<h3>ID extraction performance</h3> 
<p>The system was evaluated against 585 ID images:</p> 
<table border="1px" cellpadding="10px" class="styled-table" style="height: 196px;" width="526"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Metric</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Baseline</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>New solution</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Improvement</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Name</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">84.93%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">87.72%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">+2.79%</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Date of birth</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">81.25%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">90.80%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">+9.55%</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Document type</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">78.43%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">96.40%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">+17.97%</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">ID number</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">74.32%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">89.40%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">+15.08%</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Overall accuracy</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">79.73%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">90.80%</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">+11.07%</td> 
  </tr> 
 </tbody> 
</table> 
<p>ID number extraction, previously the weakest field at 74.32%, improved by over 15 percentage points. Document type classification reached 96.4%. Average processing time: 4.42 seconds per document.</p> 
<h3>Fraud detection performance</h3> 
<p>The combined end-to-end fraud detection pipeline (visual pattern detection plus background similarity) achieved 81% accuracy with 59% recall and 83% specificity.</p> 
<p><img alt="Performance metrics dashboard showing fraud detection system accuracy of 81%, recall of 59%, and specificity of 83%, with visual pattern detection capabilities achieving 95%+ confidence and background similarity analysis results" class="alignnone size-full wp-image-128925" height="1272" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203166.png" width="2262" /></p> 
<p><em>Fraud detection results: 81% accuracy, 59% recall, 83% specificity</em></p> 
<p>The 59% recall means the system catches about 6 in 10 fraud cases. The conservative thresholds reflect a business reality: false positives create customer friction, while missed fraud can be caught through other controls. As the fraud pattern database grows with confirmed cases, recall improves.</p> 
<h3>Cost and speed</h3> 
<p>The new solution reduced costs and processing time across both pipelines.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Component</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Cost reduction</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">ID extraction (Amazon Textract + Amazon Rekognition + Claude)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">91% reduction vs. previous solution</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Fraud detection (Claude Sonnet 4 + Amazon Titan Embeddings + Amazon S3 Vectors)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">3–5 seconds per image</td> 
  </tr> 
 </tbody> 
</table> 
<p>The ID extraction cost represents a 91% reduction from the previous solution. This makes it economically viable to serve industries with lower-value microloans. The fraud detection pipeline completes in 3–5 seconds per image.</p> 
<h3>Operational impact</h3> 
<p>Beyond accuracy and cost, the solution changed how Sun Finance operates day-to-day:</p> 
<ul> 
 <li><strong>Manual intervention</strong> projected to drop from 60% to 30% of applications, cutting the review workload in half.</li> 
 <li><strong>Staffing</strong> projected to decrease from approximately 3 FTEs to approximately 1 FTE for this industry.</li> 
 <li><strong>Region expansion</strong> now economically viable for low-value loan economies.</li> 
 <li><strong>Adaptability</strong>—adding a new document type or language requires prompt engineering and validation, not retraining specialized models.</li> 
</ul> 
<h2>Scalability and expansion</h2> 
<p>The solution’s architecture was designed for rapid expansion. Sun Finance operates across nine countries, and the serverless design enables industry-specific deployments without infrastructure duplication. Adding a new economy requires configuration updates and redeployment. The team updates Claude Sonnet 4 prompts via Amazon Bedrock and defines document-specific validation rules, then tests against a validation dataset. These configuration changes require redeploying the Lambda functions through the continuous integration and continuous delivery (CI/CD) pipeline using Terraform. The fraud detection system uses two complementary methods. Visual pattern detection via Claude Sonnet 4 identifies screen photos and digital manipulation. These techniques are largely universal across industries. Background similarity analysis using Amazon S3 Vectors catches fraud rings by comparing backgrounds against known patterns, with confirmed fraud cases added to improve detection over time.</p> 
<p>The modular architecture enables continuous enhancement. The <a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a> orchestration allows adding new fraud detection methods as parallel Lambda functions without disrupting existing checks. These could be capabilities like EXIF metadata analysis, device fingerprinting, and geolocation validation. Each would integrate as additional parallel checks without requiring architectural changes.</p> 
<h2>Lessons learned</h2> 
<p>Five practical takeaways from the engagement:</p> 
<p><strong>OCR + LLM beats LLM alone.</strong> Claude Sonnet 4 via Amazon Bedrock on its own achieved 61.8% accuracy for ID extraction, which was below the existing baseline. Adding Amazon Textract for raw text extraction and using Claude only for structuring jumped accuracy to 85%. The LLM is good at understanding context and normalizing messy data. It’s not as reliable at precise character-by-character recognition from images.</p> 
<p><strong>Multi-tier OCR delivers resilience.</strong> The cascading approach uses Amazon Textract as primary and Amazon Rekognition as a fallback. No single OCR service handled every edge case, but the combination added minimal cost while helping avoid complete failures on challenging images.</p> 
<p><strong>Fraud detection needs multiple methods.</strong> Visual pattern detection catches screen photos at 95%+ confidence. Background similarity catches fraud rings through location patterns. But background similarity only achieves 55% recall on seen patterns and drops to 16.7% on novel patterns. Neither method alone is sufficient, and the system improves as more confirmed fraud cases are added to the database.</p> 
<p><strong>Start simple, add complexity when metrics demand it.</strong> The team achieved a 91% cost reduction by using Amazon Textract as primary OCR instead of Claude for everything. They called <code>AnalyzeID</code> only when specific fields were missing and cached embeddings for fraud detection. Reserve expensive models for tasks where they’re actually needed.</p> 
<p><strong>Serverless enables rapid iteration.</strong> The parallel execution in <a href="https://aws.amazon.com/step-functions/" rel="noopener noreferrer" target="_blank">AWS Step Functions</a> cut fraud detection latency by 40% with minimal code changes. The ability to modify and deploy individual Lambda functions without downtime was critical during a 6-week engagement where the approach evolved weekly.</p> 
<h2>Next steps</h2> 
<p>Sun Finance plans to build on the proof-of-concept in several directions.</p> 
<ul> 
 <li><strong>Expand visual detection.</strong> The current system only checks for screen photos. It misses cartoons, illustrations, and AI-generated images. Expanding the detection prompt is the lowest-effort, highest-impact improvement.</li> 
 <li><strong>More training data.</strong> Continuous collection of confirmed fraud cases and diverse background patterns will directly improve background similarity recall beyond the current 55% on seen patterns.</li> 
 <li><strong>Additional fraud signals.</strong> Integrating EXIF metadata analysis, device fingerprinting, and geolocation validation would add detection paths that don’t depend on visual analysis. This is particularly valuable for novel fraud patterns.</li> 
 <li><strong>Multi-language expansion.</strong> Expanding to Sun Finance’s other economies in countries across Southeast Asia, Africa, Latin America, and Europe requires language-specific prompt engineering and validation rules. Claude’s multilingual capabilities provide a starting point, and the team is building a configuration framework to enable expansion without code changes.</li> 
</ul> 
<h2>Clean up</h2> 
<p>If you implement a similar proof-of-concept, delete the following resources when you’re done to avoid ongoing charges:</p> 
<ul> 
 <li>AWS Lambda functions created for the ID extraction and fraud detection pipelines.</li> 
 <li>AWS Step Functions state machines.</li> 
 <li>Amazon S3 buckets and Amazon S3 Vectors vector indexes used for fraud pattern storage.</li> 
 <li>Amazon API Gateway REST APIs.</li> 
 <li>Amazon Cognito user pools.</li> 
 <li>AWS WAF web access control lists (ACLs).</li> 
 <li>Any Amazon Bedrock provisioned throughput (if configured).</li> 
</ul> 
<p>You can delete these resources through the <a href="https://aws.amazon.com/console/" rel="noopener noreferrer" target="_blank">AWS Management Console</a> or by running `terraform destroy` if you deployed the infrastructure using Terraform.</p> 
<h2>Conclusion</h2> 
<p>In this post, we showed how Sun Finance combined Amazon Textract, Amazon Rekognition, and Amazon Bedrock to build an AI-powered identity verification pipeline. The solution improved extraction accuracy from 79.7% to 90.8%, cut per-document costs by 91%, and reduced processing time from up to 20 hours to under 5 seconds. The core architectural pattern, using specialized OCR for text extraction and an LLM for intelligent structuring, applies to document processing workflows where traditional OCR falls short. The serverless fraud detection system demonstrates how you can combine visual analysis with vector similarity search to catch fraud patterns at scale.</p> 
<p>For customers applying for a microloan, that’s the difference between waiting a day and getting an answer while they’re still on their phone.</p> 
<blockquote>
 <p><em>“Thank you to the AWS Generative AI Innovation Center team for an outstanding partnership and truly exceptional results. What initially felt like an ambitious — almost unrealistic — target has been transformed into a secure, production-ready solution delivering measurable gains in accuracy, speed, and cost efficiency. In particular, the AI-powered fraud detection capability — combining visual pattern recognition and background similarity analysis — represents a major step forward in protecting our portfolio while maintaining a seamless customer experience. The impact on our operations and risk management framework is immediate and significant, and we deeply appreciate the expertise, dedication, and execution excellence that made this possible.”</em></p> 
 <p>— Agris Vaselāns, Group CRO, Sun Finance</p>
</blockquote> 
<p>To learn how generative AI can improve your document processing and fraud detection workflows, visit the <a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock product page</a> or connect with the <a href="https://aws.amazon.com/ai/generative-ai/innovation-center/" rel="noopener noreferrer" target="_blank">AWS Generative AI Innovation Center</a>. For more on OCR and document processing, refer to the <a href="https://docs.aws.amazon.com/textract/latest/dg/what-is.html" rel="noopener noreferrer" target="_blank">Amazon Textract Developer Guide</a>.</p> 
<p>We’d love to hear about your experience with document processing and fraud detection. Share your thoughts in the comments section.</p> 
<hr /> 
<h2><strong>About the authors</strong></h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Babs Khalidson" class="alignleft size-full wp-image-128943" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203167.jpeg" width="100" />
  </div> 
  <p><strong>Babs Khalidson</strong> is a Deep Learning Architect at the AWS Generative AI Innovation Centre in London, where he specializes in fine-tuning large language models, building AI agents, and model deployment solutions. He has over 6 years of experience in artificial intelligence and machine learning across finance and cloud computing, with expertise spanning from research to production deployment.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Vushesh Babu Adhikari" class="alignleft size-full wp-image-128944" height="134" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203168.jpeg" width="100" />
  </div> 
  <p><strong>Vushesh Babu Adhikari</strong> is a Data scientist at the AWS Generative AI Innovation center in London with extensive expertise in developing Gen AI solutions across diverse industries. He has over 7 years of experience spanning across a diverse set of industries including Finance , Telecom , Information Technology with specialized expertise in Machine learning &amp; Artificial Intelligence.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Luisa Bertoli" class="alignleft size-full wp-image-128945" height="104" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-203169.png" width="100" />
  </div> 
  <p><strong>Luisa Bertoli</strong> is an AI Strategist at the AWS Generative AI Innovation Center. She works with large organizations on their AI strategy, adoption, and multi-year transformation plans, helping them move from experimentation to scalable, high-impact implementations. She has deep financial services domain expertise, built over years of designing and developing AI and ML products in the industry.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Kimmo Isosomppi" class="alignleft wp-image-128946 size-full" height="150" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-2031610.jpeg" width="100" />
  </div> 
  <p><strong>Kimmo Isosomppi</strong> is a Senior Solutions Architect at AWS in Helsinki, Finland. He helps enterprise customers across the Nordic and Baltic regions turn complex cloud and AI challenges into production-ready solutions, with particular expertise in generative AI, agentic AI architectures, and cloud security. He brings over two decades of experience across gaming, financial services, retail, and the public sector.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Seppo Kalliomaki" class="alignleft wp-image-128947 size-full" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-2031611.jpeg" width="100" />
  </div> 
  <p><strong>Seppo Kalliomaki</strong> is an Account Executive at AWS in Tallinn, Estonia, specializing in enterprise cloud adoption and AI transformation across the Nordic and Baltic regions. Since 2017, he has helped organizations in their cloud journey and implement generative AI solutions, with particular expertise in banking modernization, Public Sector services, and emerging AI use cases. Seppo works closely with renewing cloud strategy, migration planning, and AI adoption with AWS enterprise customers.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-129613 size-full" height="660" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/27/image-1-11.png" width="500" />
  </div> 
  <p><strong>Nicolas Metallo</strong> is a Senior Deep Learning Architect at the AWS Generative AI Innovation Center in Madrid. He designs and implements GenAI solutions using Amazon Bedrock and SageMaker, including fine-tuning LLMs, deploying multi-agent systems, and leading technical GTM for sovereign AI initiatives across EMEA.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Krišjānis Kočāns" class="alignleft wp-image-128949 size-full" height="92" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-2031613.jpeg" width="100" />
  </div> 
  <p><strong>Krišjānis Kočāns </strong> leads fraud prevention data science at Sun Finance Group across 14 countries in 4 continents, building fraud detection systems from scratch while driving Gen AI adoption.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Kaspars Magaznieks" class="alignleft wp-image-128950 size-full" height="138" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-2031614.jpeg" width="100" />
  </div> 
  <p><strong>Kaspars Magaznieks </strong>is Head of Fraud at Sun Finance – leading Fraud prevention Team, building fraud prevention framework, fraud prevention policy. Kaspars has more than 10 years’ experience in fraud prevention working in global, fast paced lending companies!</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Sergei Kiriasov" class="alignleft wp-image-128951 size-full" height="127" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/16/image-2031615.jpeg" width="100" />
  </div> 
  <p><strong>Sergei Kiriasov</strong> is Head of Risk Technology at Sun Finance, responsible for shaping and delivering the technology behind credit risk decision-making. Leading cross-functional collaboration between Risk and IT, ensures robust architecture, efficient processes, and scalable solutions that empower data science, fraud prevention, and portfolio teams. With 15+ years in technology, drives innovation and operational excellence across risk systems.</p> 
 </div> 
</footer>
