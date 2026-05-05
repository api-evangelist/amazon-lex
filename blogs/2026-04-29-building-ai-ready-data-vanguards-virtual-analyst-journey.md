---
title: "Building AI-ready data: Vanguard’s Virtual Analyst journey"
url: "https://aws.amazon.com/blogs/machine-learning/building-ai-ready-data-vanguards-virtual-analyst-journey/"
date: "Wed, 29 Apr 2026 11:56:33 +0000"
author: "Ravi Narang, Rithvik Bobbili"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p><a href="https://investor.vanguard.com/" rel="noopener" target="_blank">Vanguard</a> is a global investment management firm, offering a broad selection of investments, advice, retirement services, and insights to individual investors, institutions, and financial professionals. We operate under a unique, investor-owned structure and adhere to a straightforward purpose: To take a stand for all investors, to treat them fairly, and to give them the best chance for investing success.</p> 
<p>When Vanguard’s financial analysts needed to query complex datasets, they faced a frustrating reality: even basic questions required writing intricate SQL queries and sometimes long response times from data teams. This challenge is not unique to Vanguard: conversational AI is a scalable solution, providing analysts immediate responses. However, deploying conversational AI requires more than choosing the right foundation model—it requires AI-ready data infrastructure.</p> 
<p>In this post, you’ll learn how Vanguard built their Virtual Analyst solution by focusing on eight guiding principles of AI-ready data, the AWS services that powered their implementation, and the measurable business outcomes they achieved.</p> 
<h2>The challenge: When AI meets enterprise data complexity</h2> 
<p>Vanguard’s analysts and business stakeholders sought faster, more direct access to financial data for decision-making. The existing workflow required SQL expertise and data team support, with typical requests taking several days to fulfill. The data infrastructure required semantic context and metadata management to enable AI-powered tools to generate accurate, business-relevant insights.</p> 
<p>As the Virtual Analyst project progressed, the team discovered that building effective conversational AI wasn’t a machine learning challenge—it was a data architecture challenge. The most sophisticated foundation models require proper data foundations to deliver reliable results. This realization led to a fundamental shift in approach: instead of focusing solely on AI capabilities, Vanguard needed to build what they termed <em>AI-ready data</em>.</p> 
<h2>The collaborative imperative: Breaking down silos</h2> 
<p>Building Virtual Analyst requires something many organizations struggle with: getting traditionally siloed teams to work together. Vanguard brought together data engineers, business analysts, compliance officers, security teams, and business stakeholders. Each team brought critical expertise:</p> 
<ul> 
 <li><strong>Data engineers</strong> understood the technical infrastructure</li> 
 <li><strong>Business analysts</strong> knew the semantic meaning of financial metrics</li> 
 <li><strong>Compliance teams</strong> helped meeting regulatory requirements</li> 
 <li><strong>Business users</strong> provided the real-world context for how they are going to use the insights.</li> 
</ul> 
<p>This cross-functional collaboration became the foundation for AI by developing a well-defined, cross-functional operating model where ownership models, semantic definitions and quality standards were well understood and activated. The team realized that without clear ownership models, semantic definitions, and quality standards that all teams could understand and contribute to, the AI solution would not have a good foundation. The Virtual Analyst project served as a catalyst for new processes and frameworks that provide benefits far beyond the initial AI use case. The following figure shows the AI-ready data blueprint that was developed for the Virtual Analyst architecture.</p> 
<h2>Case Study: Virtual Analyst</h2> 
<div class="wp-caption alignnone" id="attachment_127856" style="width: 1142px;">
 <img alt="AI-Ready Data Blueprint" class="wp-image-127856" height="958" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/07/image-1-1.jpeg" width="1132" />
 <p class="wp-caption-text" id="caption-attachment-127856"><em>The architecture reflects a single, context-specific implementation, and it should be viewed as illustrative rather than prescriptive.</em></p>
</div> 
<p>Vanguard chose AWS for its&nbsp;comprehensive suite of integrated services. AWS offers a rich feature set for building AI-ready data architectures, from the advanced analytics capabilities of Amazon Redshift to the automated data cataloging on AWS Glue and the foundation model access on Amazon Bedrock. In addition, the security and compliance features of AWS met the stringent requirements of the financial services industry. The Virtual Analyst uses:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/bedrock/" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a>&nbsp;for foundation models that power natural language understanding</li> 
 <li><a href="https://aws.amazon.com/bedrock/guardrails/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Guardrails</a> to secure AI inputs and outputs to protect Vanguard’s sensitive financial data</li> 
 <li><a href="https://aws.amazon.com/ecs/" rel="noopener noreferrer" target="_blank">Amazon Elastic Container Service (Amazon ECS)</a>&nbsp;for scalable compute infrastructure</li> 
 <li><a href="https://aws.amazon.com/dynamodb/" rel="noopener noreferrer" target="_blank">Amazon DynamoDB</a>&nbsp;for conversation persistence across a horizontally scalable architecture with minimal latency</li> 
 <li><a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>&nbsp;for storage</li> 
 <li><a href="https://aws.amazon.com/sagemaker/" rel="noopener noreferrer" target="_blank">Amazon SageMaker</a>&nbsp;for experimentation</li> 
 <li><a href="https://aws.amazon.com/redshift/" rel="noopener noreferrer" target="_blank">Amazon Redshift</a>&nbsp;for centralized data warehousing</li> 
 <li><a href="https://aws.amazon.com/glue/" rel="noopener noreferrer" target="_blank">AWS Glue</a>&nbsp;for data cataloging while powering numerous extract, transform, load (ETL) jobs to consolidate accurate data</li> 
</ul> 
<h2>Eight guiding principles for AI-ready data</h2> 
<p>Through their journey building the Virtual Analyst, Vanguard identified eight guiding principles that build on existing foundational data capabilities (e.g. data platforms, integration, interoperability) and extend them to support AI-ready data. These principles emerged from real-world challenges encountered when trying to make AI systems work reliably with enterprise data at scale.</p> 
<h3>Establish clear data product and operating models</h3> 
<p>Higher quality data requires clear accountability. Data product owners are responsible for business alignment and engineering stewards should maintain technical quality. Service-level agreements (SLAs) for data freshness and reconciliation tolerance and established support models for downstream consumers will help ensure data products are reuseable, well-managed, and designed to deliver outcomes. Assign both business and technical owners to each critical data asset and document their responsibilities in writing.</p> 
<h3>Define governance and security measures</h3> 
<p>Work with your compliance and security teams early to establish enterprise identity management, role-based data access controls, query-level authorization, and retention policies. Vanguard implemented logging of authorization events to meet regulatory requirements while supporting business agility. Map your existing data access policies to the new AI system and implement row-level and column-level security where needed.</p> 
<h3>Build a metadata catalog that unifies technical and business context</h3> 
<p>Implement a unified metadata and catalog system as a control plane that centralizes both technical and business metadata while exposing them via APIs. Organizations often maintain complete technical metadata but lack integrated business context, creating misalignment between technical implementations and business requirements. Technical metadata includes table and column descriptions with data types, data lineage across transformations, synonyms and categorical indicators, and relationship mappings between datasets. Technical domain experts and data stewards define this layer. Start with your most frequently accessed datasets and systematically document their technical metadata before expanding to other data sources. Version your metadata and measure mapping accuracy to maintain discoverability and precision. Business metadata captures business definitions and rules for specific attributes, domain-specific terminology and ontologies, business ownership information, and usage context. Business users and domain experts contribute this layer through collaborative governance processes. A single catalog brings these two metadata types together, enabling AI systems to generate accurate queries that align with both technical structure and business meaning.</p> 
<h3>Implement a semantic layer to operationalize business metadata</h3> 
<p>The semantic layer operationalizes the business metadata defined in your catalog by transforming complex data structures into user-friendly formats. This implementation layer translates business definitions, rules, and ontologies into executable logic that standardizes how your organization defines key metrics and the relationships between different data elements. With this layer in place, business analysts can express their understanding of data relationships in natural language that can be interpreted and translated into structured SQL queries. By enforcing the business definitions and relationships documented in your metadata catalog, the semantic layer enhances consistency across queries, reduces the risk of errors, and streamlines SQL generation. For example, Vanguard’s semantic layer maintains the definition of <em>customer lifetime value</em> across departments and systems by implementing the business rules defined by their business users. Work with business stakeholders to document the top 20 metrics your organization uses most frequently, including their precise definitions and calculation methods.</p> 
<h3>Develop ground truth examples</h3> 
<p>Ground truth examples form another critical component, comprising a set of question-to-SQL pairs that illustrate various queries users might ask. Create a library of question-to-SQL pairs that illustrate various user queries and their correct database translations. Vanguard built a collection of over 50 exemplars that serve three purposes: few-shot prompts for the AI model (providing example question-answer pairs to guide the model’s responses), evaluation benchmarks (measuring accuracy against known correct answers), and regression testing (verifying new changes don’t break existing functionality). These examples help the AI system learn through in-context learning. Start with 20–30 examples covering your most common query patterns, then expand based on user feedback and edge cases you discover.</p> 
<h3>Implement automated data quality checks</h3> 
<p>Vanguard set up observability tools to monitor data reliability through automated checks:</p> 
<ul> 
 <li><strong>Distributional checks</strong> – Detecting anomalies in data patterns (such as sudden spikes or drops in values)</li> 
 <li><strong>Referential checks</strong> – Verifying that relationships between tables remain valid (for example, every order references a valid customer)</li> 
 <li><strong>Reconciliation checks</strong> – Confirming data consistency across systems (for example, totals match between source and warehouse)</li> 
 <li><strong>Freshness checks</strong> – Confirming data updates occur on schedule</li> 
</ul> 
<h3>Establish change control processes</h3> 
<p>Treat your semantic definitions, exemplars, and configurations as code under version control. Change control and continuous integration and deployment (CI/CD) processes treat semantic definitions, exemplars, and pipeline configurations as code under continuous integration with staged deployments and gated approvals. This approach requires stakeholder sign-off for changes that affect KPIs or SLAs while enabling safe, rapid deployment of improvements. An established change control process is essential for managing the dynamic nature of the data landscape, confirming Virtual Analyst can adapt to changes effectively. Start storing data definitions in a version control system such as Git, and require peer review before changes go to production.</p> 
<h3>Create continuous evaluation mechanisms</h3> 
<p>Finally, use continuous evaluation and improvement processes define business metrics including analyst hours saved, time-to-insight improvements, user satisfaction, and measurable revenue or profit impacts where possible. The system maintains continuous regression suites and user feedback loops to evolve examples and semantics, with automated alerts for model degradation and business impact tracking. Define 3–5 key metrics that matter to your business stakeholders and establish baseline measurements before launching your AI system.</p> 
<h2>Results: From experiment to enterprise capability</h2> 
<p>The focus on AI-ready data delivered measurable outcomes:</p> 
<ul> 
 <li>Reduced time-to-insight from days to minutes for complex financial queries with the use of the Virtual Analyst</li> 
 <li>Enabled business users to access data independently without SQL knowledge</li> 
 <li>Achieved high accuracy in AI-generated SQL queries through metadata and semantic layer implementation</li> 
 <li>Decreased data team workload for routine analytical requests</li> 
 <li>Established a reusable framework now being adopted across multiple Vanguard business units.</li> 
</ul> 
<h2>Looking forward</h2> 
<p>Vanguard is evaluating opportunities to explore how knowledge graphs and Retrieval-Augmented Generation (RAG) can further enhance Virtual Analyst. Knowledge graphs could provide explicit entity relationships, canonical resolution, and cross-domain context that materially improves fuzzy matching, join inference, and explainability for generated queries. RAG systems using <a href="https://aws.amazon.com/bedrock/knowledge-bases/" rel="noopener noreferrer" target="_blank">Amazon Bedrock Knowledge Bases</a> can use the exemplar library to increase accuracy while paving the way for intelligent feedback systems that will progressively refine model quality and reliability.</p> 
<h2>Conclusion: From AI project to data transformation</h2> 
<p>In this post, we showed you how Vanguard established new standards and ways of working that began a transformation of its data analytics capabilities, leveraging data as a strategic asset. What began as an AI project revealed the groundwork an organization needs to enable AI capabilities, as shown with these eight guiding principles. Successful AI isn’t just about better algorithms—it’s about building better data foundations to support AI at enterprise scale. The combination of the integrated data and AI services of AWS, coupled with disciplined data product practices, helps organizations convert model capabilities into dependable business outcomes that executives can trust for critical decision making.</p> 
<hr /> 
<h2>About Authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft wp-image-127860" height="98" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/07/Adobe-Express-file-100x98.png" width="100" />
  </div> 
  <h3 class="lb-h4">Ravi Narang</h3> 
  <p>Ravi Narang is a data and AI leader with over 25 years of experience in artificial intelligence, machine learning, and data engineering. As Head of AI/ML Engineering at Vanguard, he leads the design and development of advanced AI and generative AI solutions that power intelligent decision-making across institutional and advisory domains. His expertise spans data readiness, semantic modeling, large language model operations, and agentic AI systems, focusing on building scalable, trustworthy, and high-impact AI systems.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-127857 size-thumbnail" height="101" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/07/image-3-1-e1775579889723-100x101.jpeg" width="100" />
  </div> 
  <h3 class="lb-h4">Rithvik Bobbili</h3> 
  <p>Rithvik Bobbili is a Machine Learning Engineer Specialist within the Center for Analytics and Insights at Vanguard. He has been at Vanguard for over two years and has supported numerous AI/ML initiatives powered by both traditional machine learning as well as the latest advancements in generative AI. He specializes in designing generative AI solutions to solve business problems, working with LLMs, agents, and more to deliver innovative solutions that drive business value.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-127858 size-full" height="114" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/07/image-4-3.png" width="100" />
  </div> 
  <h3 class="lb-h4">Jiwon Yeom</h3> 
  <p>Jiwon Yeom is a Solutions Architect at AWS, based in New York City. She focuses on generative AI in the financial services industry and is passionate about helping customers build scalable, secure, and human-centered AI solutions. Outside of work, she enjoys writing and exploring hidden bookstores.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-127859 size-thumbnail" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/07/image-5-3-100x100.png" width="100" />
  </div> 
  <h3 class="lb-h4">Matt Lanza</h3> 
  <p>Matt Lanza is a Principal Solutions Architect at AWS. He is interested in helping customers build resilient architecture on AWS. He drives fast when he gets a chance.</p> 
 </div> 
</footer> 
<p>© [2026] <em>The Vanguard Group, Inc. All rights reserved. This material is provided for informational purposes only and is not intended to be investment advice or a recommendation to take any particular investment action.</em></p>
