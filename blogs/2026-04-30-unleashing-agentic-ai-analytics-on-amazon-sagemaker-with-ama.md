---
title: "Unleashing Agentic AI Analytics on Amazon SageMaker with Amazon Athena and Amazon Quick"
url: "https://aws.amazon.com/blogs/machine-learning/unleashing-agentic-ai-analytics-on-amazon-sagemaker-with-amazon-athena-and-amazon-quick/"
date: "Thu, 30 Apr 2026 16:52:40 +0000"
author: "Raj Balani"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p>Modern enterprises face mounting challenges in extracting actionable insights from vast data lakes and lakehouses spanning petabytes of structured and unstructured data. Traditional analytics require specialized technical expertise in SQL, data modeling, and business intelligence tools, creating bottlenecks that slow decision-making across retail, financial services, healthcare, Travel &amp; Hospitality, manufacturing and many more industries. This architecture demonstrates how agentic AI assistant from <a href="https://aws.amazon.com/quick/" rel="noopener noreferrer" target="_blank">Amazon Quick</a> transform data analytics into a self-service capability. It showcases enabling business users to query complex structured datasets and mix with unstructured data to find the valuable insights to improve their business outcomes through intuitive natural language interfaces.</p> 
<p>To demonstrate the functionality, we built a lakehouse using the TPC-H datasets as our foundation. This integrated architecture leverages <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) as a storage, <a href="https://aws.amazon.com/sagemaker/" rel="noopener noreferrer" target="_blank">Amazon SageMaker</a> and <a href="https://aws.amazon.com/sagemaker/catalog/" rel="noopener noreferrer" target="_blank">AWS Glue</a> for lakehouse, <a href="https://aws.amazon.com/athena/" rel="noopener noreferrer" target="_blank">Amazon Athena</a> for serverless SQL querying across multiple storage formats (S3 Table, Iceberg, and Parquet), and multiple features from Quick to build dashboard and conversational AI agents that provide natural language access to data insights. Through integrated knowledge bases using <a href="https://aws.amazon.com/quick/spaces/" rel="noopener noreferrer" target="_blank">Amazon Quick spaces</a>, this solution democratizes lakehouse data access for business users while preserving enterprise-grade security, governance frameworks, and the scalability required for modern data-driven decision-making across the organization.</p> 
<h2>Solution Overview</h2> 
<p>The following diagram shows the overall design and corresponding dataflow that we implemented as part of this blog post.</p> 
<div class="wp-caption alignnone" id="attachment_128264" style="width: 2964px;">
 <img alt="AWS data analytics architecture diagram showing data flow from TPC-H structured data through Amazon SageMaker, S3, Athena, Quick Sight to end users, with numbered workflow steps 1-9" class="wp-image-128264 size-full" height="1406" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/Screenshot-2026-04-09-at-1.56.41 PM.png" width="2954" />
 <p class="wp-caption-text" id="caption-attachment-128264">Figure 1: Overall design diagram Reference following steps for the detailed end to end data flow and user interaction capabilities.</p>
</div> 
<ol> 
 <li>Data Source Ingestion: Structured Data TPC-H serves as the primary data source, containing benchmark datasets stored in relational database format. AWS hosted the TPC-H data in the publicly available s3 bucket (<a href="https://s3://redshift-downloads/TPC-H/2.18/100GB" rel="noopener noreferrer" target="_blank">s3://redshift-downloads/TPC-H/2.18/100GB</a>)</li> 
 <li>Data Load: Amazon Athena performs the first query layer, executing serverless SQL queries against the TPC-H structured data to extract, prepare data for processing, load data in S3, and create corresponding catalog in Glue.</li> 
 <li>Multi-Format Storage Layer: To illustrate the versatility of Data lake and Lakehouse we saved the data into three optimized storage formats: 
  <ol type="a"> 
   <li>Amazon S3 -CSV: Use external table to create Athena table based on existing CSV files.</li> 
   <li>Amazon S3 (<a href="https://iceberg.apache.org/" rel="noopener noreferrer" target="_blank">Apache Iceberg-parquet</a>): ACID-compatible table format enabling time-travel and schema evolution</li> 
   <li><a href="https://aws.amazon.com/s3/features/tables/" rel="noopener noreferrer" target="_blank">Amazon S3 Table</a>: Amazon S3 Tables deliver the first cloud object store with built-in Apache Iceberg support and streamline storing tabular data at scale.</li> 
  </ol> </li> 
 <li>Metadata Cataloging: AWS Glue Catalog indexes all three storage formats, creating a unified metadata layer that enables seamless querying across different data formats.</li> 
 <li>Lakehouse Query Layer: We used the Amazon Athena SQL queries across storage formats (S3 Table, Iceberg, and Parquet) using the Glue Catalog metadata, providing a unified query interface.</li> 
 <li>Business Intelligence Pipeline: Structured TPC-H data flows into Amazon Quick, which integrates with Quick Sight to create: 
  <ol type="a"> 
   <li>Dataset – We utilized <a href="https://docs.aws.amazon.com/quick/latest/userguide/athena.html" rel="noopener noreferrer" target="_blank">Amazon Athena connection from Amazon Quick</a> to extract structured data to load in <a href="https://docs.aws.amazon.com/quick/latest/userguide/spice.html" rel="noopener noreferrer" target="_blank">Quick SPICE (Super-fast, Parallel, In-memory Calculation Engine) dataset</a></li> 
   <li>Topic – Organized data domains for business context</li> 
   <li>Dashboard Using Q – Interactive visualizations with natural language query capabilities to build the dashboard and publish it</li> 
  </ol> </li> 
 <li>AI Knowledge Enhancement: Parallel to the structured data flow, a Web Crawler for <a href="https://www.tpc.org/tpc_documents_current_versions/pdf/tpc-h_v2.17.1.pdf" rel="noopener noreferrer" target="_blank">TPC-H specifications ingests unstructured data (documentation, specifications)</a> and feeds it into Knowledge Bases to provide contextual understanding.</li> 
 <li>Conversational Agentic AI Layer: Knowledge Bases power <a href="https://aws.amazon.com/quick/spaces/" rel="noopener noreferrer" target="_blank">Amazon Quick spaces</a> (collaborative environments), which in turn enable the <a href="https://aws.amazon.com/quick/chat-agents/" rel="noopener noreferrer" target="_blank">Amazon Quick chat agents</a> with contextual awareness and domain knowledge for natural language interactions.</li> 
 <li>End User Access: Users interact with the system through two primary interfaces: 
  <ol type="a"> 
   <li>Dashboard Using Q – Visual analytics and self-service Business Intelligence</li> 
   <li>Chat Agent – Conversational AI for natural language data exploration</li> 
  </ol> </li> 
</ol> 
<h2>Pre-requisite</h2> 
<p>Before you get started, make sure you have the following prerequisites:</p> 
<ul> 
 <li>An <a href="http://console.aws.amazon.com/" rel="noopener noreferrer" target="_blank">AWS account</a> and <a href="https://docs.aws.amazon.com/quick/latest/userguide/setting-up.html" rel="noopener noreferrer" target="_blank">Amazon Quick account</a></li> 
 <li>A basic understanding of <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service,</a> <a href="https://aws.amazon.com/sagemaker/" rel="noopener noreferrer" target="_blank">Amazon SageMaker,</a> <a href="https://aws.amazon.com/lake-formation/" rel="noopener noreferrer" target="_blank">AWS Lake Formation</a>, and <a href="https://aws.amazon.com/athena/" rel="noopener noreferrer" target="_blank">Amazon Athena</a></li> 
 <li>Console role with permissions to create the dataset in S3, run Athena queries, create Glue catalog, Lake Formation admin privileges, and access Quick features. To decide the relevant policy/policies <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies" rel="noopener noreferrer" target="_blank">reference policy document.</a></li> 
</ul> 
<h2>Data Preparation for lakehouse / data Lake</h2> 
<p>In this section, we will mimic many of the <strong>data lake</strong> features by working with <strong>external tables</strong>, which allow querying data stored in Amazon S3 without loading it into a managed storage layer. We will explore <strong>Open Table Format (OTF)</strong> tables using Apache Iceberg to consider possible ACID transactions supported tables. Amazon managed <strong>S3 Tables</strong> will be leveraged to showcase how Amazon natively supports Iceberg-compatible table management directly within S3, simplifying lakehouse architecture at scale. Throughout these exercises, we will use the industry-standard <strong>TPC-H dataset</strong>, a benchmark workload representing a realistic business data model with orders, customers, and line items to make sure our examples are both meaningful and reproducible.</p> 
<p>We will leverage Amazon Athena for data preparation. If this is your first time using <strong>Amazon Athena</strong>, you’ll need to create an <strong>Amazon S3 bucket</strong> to store your query results. Athena uses S3 as its output location before you can run queries. Follow the official AWS getting started guide to complete this one-time setup: <a href="https://docs.aws.amazon.com/athena/latest/ug/getting-started.html" rel="noopener noreferrer" target="_blank">Getting Started with Amazon Athena.</a> Alternately, you can use <a href="https://docs.aws.amazon.com/athena/latest/ug/managed-results.html" rel="noopener noreferrer" target="_blank">Managed query results</a> feature.</p> 
<p><strong>Tip:</strong> Choose an S3 bucket in the <strong>same AWS Region</strong> as your data sources to avoid cross-region data transfer costs and latency.</p> 
<p>Once your S3 output location is configured, you’re ready to proceed.</p> 
<h3>Create the Glue Database</h3> 
<p>Start by creating a Glue database that will serve as the metadata catalog for all your tables using Athena. Run the following SQL in the Athena query editor:</p> 
<pre><code class="language-sql">CREATE DATABASE IF NOT EXISTS blog_qs_athena_tpc_h_db_sql COMMENT 'TPC-H database'; </code></pre> 
<div class="wp-caption alignnone" id="attachment_128221" style="width: 1441px;">
 <img alt="Amazon Athena Query Editor interface showing SQL query to create TPC-H database, with completed execution status showing 82ms queue time, 230ms run time, and primary workgroup selected" class="wp-image-128221 size-full" height="447" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-2.png" width="1431" />
 <p class="wp-caption-text" id="caption-attachment-128221">Figure 2: Database creation blog_qs_athena_tpc_h_db_sql</p>
</div> 
<p><strong>What this does:</strong> This registers a logical database in the AWS Glue Data Catalog, which Athena uses to organize and discover your tables. Tables created in subsequent steps will live under this database.</p> 
<h3>Create an External Table on S3</h3> 
<p>Next, create an external table pointing to the TPC-H “customer” dataset stored in a public S3 bucket (<code>'s3://redshift-downloads/TPC-H/2.18/100GB/customer/'</code>). External tables in Athena don’t move or copy data — they query it directly from S3, making this a fast and cost-effective way to explore raw data.</p> 
<pre><code class="language-sql">CREATE EXTERNAL TABLE IF NOT EXISTS blog_qs_athena_tpc_h_db_sql.customer_csv 
( 
	C_CUSTKEY INT, 
	C_NAME STRING, 
	C_ADDRESS STRING, 
	C_NATIONKEY INT, 
	C_PHONE STRING, 
	C_ACCTBAL DOUBLE, 
	C_MKTSEGMENT STRING, 
	C_COMMENT STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '|'
STORED AS TEXTFILE
LOCATION 's3://redshift-downloads/TPC-H/2.18/100GB/customer/'
TBLPROPERTIES ('classification' = 'csv'); </code></pre> 
<p>Verify the table by previewing a few rows:</p> 
<pre><code class="language-sql">SELECT * FROM blog_qs_athena_tpc_h_db_sql.customer_csv LIMIT 10; </code></pre> 
<div class="wp-caption alignnone" id="attachment_128222" style="width: 1440px;">
 <img alt="" class="wp-image-128222 size-full" height="582" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-3.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-128222">Figure 3: verify blog_qs_athena_tpc_h_db_sql.customer_csv</p>
</div> 
<h3>Create an Apache Iceberg Table</h3> 
<p>Next, we will mimic the table using Apache Iceberg, which is an open table format that brings ACID transactions, time travel, and partition evolution to your data lake — making it ideal for production-grade workloads. This is a three-step process.</p> 
<p><strong>Step1: Create the S3 Bucket </strong>– Before writing SQL queries, set up your storage layer. You can create an S3 bucket using the <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html" rel="noopener noreferrer" target="_blank">AWS Management Console or AWS CLI</a>.</p> 
<p><strong>For this blog, I’m using the S3 bucket</strong>: <code>amzn-s3-demo-bucket</code></p> 
<p><strong>Note</strong>: Your bucket name will be different, as S3 bucket names must be globally unique across all AWS accounts.</p> 
<p><strong>Step2: Create an External CSV Table for Orders </strong>– First, register the raw orders data as an external table in its original format, in our case it’s CSV.</p> 
<pre><code class="language-sql">CREATE EXTERNAL TABLE IF NOT EXISTS blog_qs_athena_tpc_h_db_sql.orders_csv 
( 
	O_ORDERKEY BIGINT, 
	O_CUSTKEY BIGINT, 
	O_ORDERSTATUS STRING, 
	O_TOTALPRICE DOUBLE, 
	O_ORDERDATE STRING, 
	O_ORDERPRIORITY STRING, 
	O_CLERK STRING, 
	O_SHIPPRIORITY INT, 
	O_COMMENT STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES ('field.delim' = '|')
LOCATION 's3://redshift-downloads/TPC-H/2.18/100GB/orders/'
TBLPROPERTIES ('classification' = 'csv');</code></pre> 
<p>Let’s verify the dataset.</p> 
<pre><code class="language-sql">SELECT * FROM blog_qs_athena_tpc_h_db_sql.orders_csv LIMIT 10;</code></pre> 
<div class="wp-caption alignnone" id="attachment_128223" style="width: 1439px;">
 <img alt="" class="wp-image-128223 size-full" height="555" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-4.png" width="1429" />
 <p class="wp-caption-text" id="caption-attachment-128223">Figure 4: verify blog_qs_athena_tpc_h_db_sql.orders_csv</p>
</div> 
<p><strong>Step3: Create the Iceberg Table Using CREATE TABLE AS SELECT (CTAS) – </strong>Use <strong>CREATE TABLE AS SELECT (CTAS)</strong> to create a self-managed Iceberg table in Parquet format, partitioned by order date. We’ll load a sample date range O_ORDERDATE BETWEEN ‘1998-06-01’ AND ‘1998-12-31’.</p> 
<pre><code class="language-sql">CREATE TABLE blog_qs_athena_tpc_h_db_sql.orders_iceberg
WITH ( 
	table_type = 'ICEBERG', 
	format = 'PARQUET', 
	is_external = false, 
	partitioning = ARRAY['o_orderdate'], 
	location = 's3://amzn-s3-demo-bucket/tpch_iceberg/orders/')
AS
SELECT * FROM blog_qs_athena_tpc_h_db_sql.orders_csv
WHERE O_ORDERDATE BETWEEN '1998-06-01' AND '1998-12-31';</code></pre> 
<p>Verify the Iceberg table data:</p> 
<pre><code class="language-sql">SELECT * FROM blog_qs_athena_tpc_h_db_sql.orders_iceberg LIMIT 10; </code></pre> 
<div class="wp-caption alignnone" id="attachment_128224" style="width: 1440px;">
 <img alt="" class="wp-image-128224 size-full" height="655" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-5.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-128224">Figure 5: verify blog_qs_athena_tpc_h_db_sql.orders_iceberg</p>
</div> 
<h3>Create an Amazon S3 Table</h3> 
<p>Amazon S3 Tables are purpose-built, fully managed tables with built-in Apache Iceberg support. It delivers high-performance query throughput without the overhead of managing maintenance operations, such as compaction, snapshot management, and unreferenced file removal. This is a three-step process.</p> 
<p><strong>Step1: Create the S3 Table Bucket and Namespace – </strong>Navigate to <strong>S3 → Table Buckets</strong> in the AWS Console to create the bucket <code>blog-qs-athena-tpc-h-db-sql-s3-table-mar-3</code> and namespace. Alternatively, use the AWS <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-namespace-create.html" rel="noopener noreferrer" target="_blank">CLI for scripted setup</a>.</p> 
<p>Note : You can ignore these steps if you already have an S3 table bucket and namespace available.</p> 
<div class="wp-caption alignnone" id="attachment_128225" style="width: 1439px;">
 <img alt="" class="wp-image-128225 size-full" height="672" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-6.png" width="1429" />
 <p class="wp-caption-text" id="caption-attachment-128225">Figure 6: Create S3 Table bucket blog-qs-athena-tpc-h-db-sql-s3-table-mar-3</p>
</div> 
<p>Now let’s create a namespace <code>blog_qs_athena_tpc_h_namespace </code>associated with above S3 table bucket by clicking on the <code>blog-qs-athena-tpc-h-db-sql-s3-table-mar-3</code>.</p> 
<div class="wp-caption alignnone" id="attachment_128226" style="width: 1482px;">
 <img alt="" class="wp-image-128226 size-full" height="786" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-7.png" width="1472" />
 <p class="wp-caption-text" id="caption-attachment-128226">Figure 7: Create S3 table Namespace blog_qs_athena_tpc_h_namespace</p>
</div> 
<p><strong>Step2: Create an External CSV Table for Line Items – </strong>Use Athena to register the TPC-H line items dataset as an external table:</p> 
<pre><code class="language-sql">CREATE EXTERNAL TABLE IF NOT EXISTS blog_qs_athena_tpc_h_db_sql.lineitem_csv 
( 
	L_ORDERKEY BIGINT, 
	L_PARTKEY BIGINT, 
	L_SUPPKEY BIGINT, 
	L_LINENUMBER INT, 
	L_QUANTITY DECIMAL(15,2), 
	L_EXTENDEDPRICE DECIMAL(15,2), 
	L_DISCOUNT DECIMAL(15,2), 
	L_TAX DECIMAL(15,2), 
	L_RETURNFLAG STRING, 
	L_LINESTATUS STRING, 
	L_SHIPDATE STRING, 
	L_COMMITDATE STRING, 
	L_RECEIPTDATE STRING, 
	L_SHIPINSTRUCT STRING, 
	L_SHIPMODE STRING, 
	L_COMMENT STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '|'
STORED AS TEXTFILE
LOCATION 's3://redshift-downloads/TPC-H/2.18/100GB/lineitem/'
TBLPROPERTIES ('skip.header.line.count' = '0');</code></pre> 
<p>Preview the data:</p> 
<pre><code class="language-sql">SELECT * FROM blog_qs_athena_tpc_h_db_sql.lineitem_csv LIMIT 10;</code></pre> 
<div class="wp-caption alignnone" id="attachment_128227" style="width: 1440px;">
 <img alt="" class="wp-image-128227 size-full" height="650" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-8.png" width="1430" />
 <p class="wp-caption-text" id="caption-attachment-128227">Figure 8: verify data blog_qs_athena_tpc_h_db_sql.lineitem_csv</p>
</div> 
<p><strong>Step3: Create the S3 Tables Table Using CTAS – </strong>Finally, create a Parquet-formatted S3 Tables in your new catalog using CTAS. We filter a sample date range to limit the initial data load based on CAST(L_SHIPDATE AS DATE) BETWEEN DATE(‘1998-06-01’) AND DATE(‘1998-12-31’).</p> 
<p>Note: Make sure to use <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/create-s3-tables-catalog.html" rel="noopener noreferrer" target="_blank">s3tablescatalog</a> to run the following queries as shown in the following screenshot.</p> 
<pre><code class="language-sql">CREATE TABLE lineitem_csv_s3_table
WITH ( format = 'PARQUET')
AS
SELECT * FROM AwsDataCatalog.blog_qs_athena_tpc_h_db_sql.lineitem_csv
WHERE CAST(L_SHIPDATE AS DATE) BETWEEN DATE('1998-06-01') AND DATE('1998-12-31');</code></pre> 
<p>Verify the result:</p> 
<pre><code class="language-sql">SELECT * FROM lineitem_csv_s3_table LIMIT 10;</code></pre> 
<div class="wp-caption alignnone" id="attachment_128228" style="width: 1441px;">
 <img alt="" class="wp-image-128228 size-full" height="676" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-9.png" width="1431" />
 <p class="wp-caption-text" id="caption-attachment-128228">Figure 9: verify data lineitem_csv_s3_table</p>
</div> 
<h2>Dataset Preparation in Amazon Quick</h2> 
<p>Your Athena tables are registered and queryable. Now it is time to bring that data into Amazon Quick – connecting it, shaping it, and making it speak the language of your business. This section walks through every step: connecting to the Athena data source, creating datasets and importing them into SPICE, joining the three SPICE datasets, configuring a Quick Topic for natural language Q&amp;A, building and publishing a dashboard with Amazon Q, and setting up the Knowledge Base that powers the agentic layer.</p> 
<h3>Data Source Creation</h3> 
<p>Before Amazon Quick can query your three tables in your data lake, you create a single Athena data source connection. You can access all three tables — the CSV external table, the self-managed Iceberg Parquet table, and the S3 Tables managed Iceberg table — using the same connection because all three are cataloged in AWS Glue Data Catalog and accessible through the same Athena workgroup.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Datasets → Data sources →Create data source</strong>.</li> 
 <li>Select <strong>Amazon Athena</strong> as the data source type.</li> 
 <li>Enter a descriptive name (for example <code>tpch-lakehouse-athena</code>).</li> 
 <li>Select the Athena <strong>workgroup</strong> your team uses for production queries. Using a dedicated workgroup enforces query cost controls and separates Quick query traffic from other workloads.</li> 
 <li>Choose <strong>Validate connection</strong>. Quick confirms it can reach Athena and the Glue Data Catalog.</li> 
 <li>Select <strong>Create data source</strong>.</li> 
</ol> 
<h3>Dataset Creation and SPICE Ingestion</h3> 
<p>With the Athena data source created, create one Quick dataset per table. Import each dataset into <strong>SPICE</strong> — Quick’s Super-fast, Parallel, In-memory Calculation Engine — to deliver sub-second query performance in dashboards and agentic workflows, regardless of how large the underlying S3 data grows.</p> 
<h4>Lake Formation Permissions</h4> 
<p>Before creating datasets, make sure the appropriate data access permissions are in place:</p> 
<ul> 
 <li><strong>If Lake Formation is not enabled:</strong> Permissions are managed at the Quick service role level via standard IAM-based S3 access control. Make sure the Quick service role (for example, <code>aws-quicksight-service-role-v0</code>) has the read IAM permissions for the relevant S3 buckets and Athena resources. No additional Lake Formation configuration is required.</li> 
 <li><strong>If Lake Formation is enabled:</strong> Lake Formation acts as the central authorization layer, overriding standard IAM-based S3 permissions. Grant permissions directly to the Amazon Quick author or IAM role: 
  <ul> 
   <li>Open the <a href="https://console.aws.amazon.com/lakeformation/" rel="noopener noreferrer" target="_blank">AWS Lake Formation console</a>.</li> 
   <li>Choose <strong>Permissions → Data permissions → Grant</strong>.</li> 
   <li>Select the SAML users and groups.</li> 
   <li>Enter Quick user ARN</li> 
   <li>Choose <strong>Named Data Catalog resources</strong></li> 
  </ul> </li> 
</ul> 
<p><img alt="" class="alignnone wp-image-128229 size-full" height="680" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-10.png" width="750" /></p> 
<p>Figure 10: Lake Formation permissions</p> 
<ul> 
 <li> 
  <ul> 
   <li>Choose the required databases, tables, and columns.</li> 
   <li>Grant <strong>SELECT</strong> at minimum; add <strong>DESCRIBE</strong> for dataset creation.</li> 
   <li>Repeat for each user or role that requires access.</li> 
  </ul> </li> 
</ul> 
<p>For step-by-step instructions, see <a href="https://docs.aws.amazon.com/quick/latest/userguide/lake-formation.html" rel="noopener noreferrer" target="_blank">Securely analyze your data with AWS Lake Formation and Amazon Quick Sight,</a> and <a href="https://builder.aws.com/content/3B8KSSr0Z4DjwG3l2onzdGrnVU9/accessing-amazon-s3-tables-through-amazon-quick-with-aws-lake-formation-permissions" rel="noopener noreferrer" target="_blank">Accessing Amazon S3 Tables through Amazon Quick with AWS Lake Formation Permissions.</a></p> 
<p>For S3 Tables specifically, the Quick service role also requires an additional <code>glue:GetCatalog</code> inline policy to access the non-default s3tablescatalog catalog — see <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-integrating-quicksight.html" rel="noopener noreferrer" target="_blank">Visualizing S3 table data with Amazon Quick</a> for the exact policy statement.</p> 
<h4>Dataset 1 — CSV External Table (customer_csv)</h4> 
<ol> 
 <li>From the Athena data source, choose <strong>Create dataset</strong>.</li> 
 <li>Select the Glue database and choose the table (for example <code>customer_csv</code>).</li> 
 <li>Select <strong>Edit/Preview data</strong> to open the data preparation experience.</li> 
 <li>Verify column data types and make changes as needed. <strong>Note:</strong> If you are using the new data preparation experience, click the <strong>Preview</strong> tab to review the data before proceeding.</li> 
 <li>Set <strong>Query mode</strong> to <strong>SPICE</strong>.</li> 
 <li>Name the dataset <code>TPC-H Customer (CSV)</code>&nbsp;and select <strong>Save &amp; publish</strong>.</li> 
</ol> 
<h4>Dataset 2 — Self-Managed Iceberg Parquet (orders_iceberg)</h4> 
<ol> 
 <li>From the same Athena data source, choose <strong>Create dataset</strong>.</li> 
 <li>Select the Glue database and choose the table (for example <code>orders_iceberg</code><em>)</em>.</li> 
 <li>Select <strong>Edit/Preview data</strong> to open the data preparation experience.</li> 
 <li>Verify column data types and make changes as needed. <strong>Note:</strong> If you are using the new data preparation experience, click the <strong>Preview</strong> tab to review the data before proceeding.</li> 
 <li>Set <strong>Query mode</strong> to <strong>SPICE</strong>.</li> 
 <li>Name the dataset <code>TPC-H Orders (Iceberg)</code>&nbsp;and select <strong>Save &amp; publish</strong>.</li> 
</ol> 
<h4>Dataset 3 — S3 Tables Managed Iceberg (lineitem_csv_s3_table)</h4> 
<p>S3 Tables are stored in a non-default AWS Glue catalog (<code>s3tablescatalog</code>), not in the standard AWSDataCatalog. Because of this, <strong>the Quick visual table browser cannot display S3 Tables</strong> — they do not appear in the “Choose your table” pane. You must use <strong>Custom SQL</strong> to query S3 Tables data and create a Quick dataset from it.</p> 
<ol> 
 <li>From the same Athena data source, choose <strong>Create dataset</strong>.</li> 
 <li>Select <strong>Use custom SQL</strong>.</li> 
 <li>Select <strong>Edit/Preview data</strong> to open the data preparation experience.</li> 
 <li>Enter an Athena SQL query referencing the S3 Tables catalog using the <code><em>“s3tablescatalog/&lt;table-bucket-name&gt;”.”&lt;namespace&gt;”.”&lt;table-name&gt;”</em> </code>syntax:</li> 
</ol> 
<pre><code class="language-sql">SELECT * FROM "s3tablescatalog/blog-qs-athena-tpc-h-db-sql-s3-table-mar-3"."blog_qs_athena_tpc_h_namespace"."lineitem_csv_s3_table"</code></pre> 
<ol start="5"> 
 <li>Choose <strong>Apply</strong>. Quick executes the query through Athena and previews the result set.</li> 
</ol> 
<div class="wp-caption alignnone" id="attachment_128230" style="width: 3006px;">
 <img alt="" class="wp-image-128230 size-full" height="1398" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-11.png" width="2996" />
 <p class="wp-caption-text" id="caption-attachment-128230">Figure 11: Preview S3 Table data from Quick</p>
</div> 
<ol start="6"> 
 <li>Verify column data types and make changes as needed.</li> 
 <li>Set <strong>Query mode</strong> to <strong>SPICE</strong>.</li> 
 <li>Name the dataset <code>TPC-H Lineitem (S3 Tables)</code>&nbsp;and select <strong>Save &amp; publish</strong>.</li> 
</ol> 
<p><strong>Note:</strong> This custom SQL requirement applies specifically to S3 Tables because they reside in a child Glue catalog registered separately from the default AWSDataCatalog. The CSV and Iceberg tables in the standard catalog are visible in the table browser and do not require custom SQL.</p> 
<h2>Joining Datasets</h2> 
<p>The TPC-H schema is a star schema by design, and Amazon Quick’s visual data preparation experience supports joining datasets directly in the UI. In this solution, we will pre-join all three tables in Athena using Custom SQL and ingest the unified result directly into SPICE as a single flat dataset. This removes Quick’s secondary table size constraint entirely and delegates the join to Athena, which handles tables of varying scale.</p> 
<p><strong>Note on the cross-source JOIN limit:</strong> If your secondary tables (<code>orders_iceberg</code> + <code>customer_csv</code>) are small enough to fit under 1 GB combined, you can perform the join inside Quick’s visual data preparation experience by opening the largest table first (making it the primary) and adding the smaller tables as secondary joins. For large TPC-H scale factors where the <code>lineitem</code> table dominates, the Athena pre-join approach below is the recommended path.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>From the Athena data source, choose <strong>Create dataset</strong>.</li> 
 <li>Select <strong>Use custom SQL</strong>.</li> 
 <li>Select <strong>Edit/Preview data</strong> to open the data preparation experience.</li> 
 <li>Enter the following Athena SQL query, which joins all three tables across the default Glue catalog (<code>blog_qs_athena_tpc_h_db_sql</code>) and the S3 Tables non-default catalog (<code>s3tablescatalog</code>):</li> 
</ol> 
<pre><code class="language-sql">SELECT 
	c.c_custkey, 
	c.c_name, 
	c.c_mktsegment, 
	c.c_nationkey, 
	o.o_orderkey, 
	o.o_orderdate, 
	o.o_orderstatus, 
	o.o_totalprice, 
	o.o_orderpriority, 
	l.l_linenumber, 
	l.l_partkey, 
	l.l_suppkey, 
	l.l_quantity, 
	l.l_extendedprice, 
	l.l_discount, 
	l.l_shipmode, 
	l.l_returnflag
FROM "s3tablescatalog/blog-qs-athena-tpc-h-db-sql-s3-table-mar-3"."blog_qs_athena_tpc_h_namespace"."lineitem_csv_s3_table" l
INNER JOIN "blog_qs_athena_tpc_h_db_sql"."orders_iceberg" o 
	ON l.l_orderkey = o.o_orderkey
INNER JOIN "blog_qs_athena_tpc_h_db_sql"."customer_csv" c 
	ON o.o_custkey = c.c_custkey; </code></pre> 
<p>The query joins the three tables using the TPC-H foreign key relationships:</p> 
<ul> 
 <li><code>lineitem_csv_s3_table.l_orderkey = orders_iceberg.o_orderkey</code> (Lineitem → Orders)</li> 
 <li><code>orders_iceberg.o_custkey = customer_csv.c_custkey</code> (Orders → Customer)</li> 
</ul> 
<p><strong>Tip:</strong> Use explicit double quotes around both the database and table names in Athena SQL — this helps prevent parse errors caused by hyphens or other special characters in identifier names, particularly for S3 Tables catalog paths.</p> 
<ol> 
 <li>Choose <strong>Apply</strong>. Quick executes the query through Athena and previews the unified result set.</li> 
 <li>Verify column data types and make changes as needed. Hide internal key columns (<code>c_custkey</code>, <code>o_custkey</code>, <code>o_orderkey</code>, <code>l_orderkey</code>) that business users do not need to see in dashboards or Q&amp;A.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-128231 size-full" height="1398" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-12.png" width="3016" />Figure 12 : Preview denormalized data from Quick</p> 
<ol start="3"> 
 <li>Set <strong>Query mode</strong> to <strong>SPICE</strong>.</li> 
 <li>Name the dataset <code>TPC-H Unified (Joined)</code>&nbsp;and select <strong>Save &amp; publish </strong>and wait for the SPICE dataset status change to “Ready” (expected time 2-3 mins)</li> 
</ol> 
<p>The joined dataset is now a single, denormalized SPICE dataset combining customer, order, and line item data across all three table formats — CSV external, self-managed Iceberg Parquet, and S3 Tables managed Iceberg — ready for both dashboard authoring and natural language Q&amp;A.</p> 
<h2>Quick Topic Configuration</h2> 
<p>A Quick Topic is the semantic layer that translates column names into business concepts. When a user asks <em>“What was total revenue last quarter by customer segment?”</em>, the <strong>Topic</strong> maps <code>revenue</code> to <code>l_extendedprice</code>, “last quarter” to a date filter on <code>o_orderdate</code>, and <code>customer segment</code> to <code>c_mktsegment</code>. Without a well-configured Topic, natural language queries return generic or incorrect results. With one, they return precise, cited answers in seconds.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Topics → Create topic</strong>.</li> 
 <li>Enter a name <code>TPC-H Analytics</code>&nbsp;and a plain-language description: <em>“Customer, order, and line item data from the TPC-H benchmark dataset, covering revenue, pricing, discounts, order status, and customer market segments.”</em></li> 
 <li>Select the <code>TPC-H Unified</code> (Joined) dataset as the data source.</li> 
 <li>Quick analyzes the dataset and auto-generates field configurations (expected time to complete 8-10 min). Review each field on the <strong>Data</strong> tab:</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-128232 size-full" height="693" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-13.png" width="1835" />Figure 13: Quick Topic enhancement</p> 
<ol start="5"> 
 <li>Add <strong>named entities</strong> for common business groupings.</li> 
 <li>Add <strong>suggested questions</strong> to guide first-time users: 
  <ol type="a"> 
   <li><em>“What is total revenue by order status this year?”</em></li> 
   <li><em>“Which customer segments placed the most orders last quarter?”</em></li> 
   <li><em>“Show me the top 10 orders by total price last month.”</em></li> 
  </ol> </li> 
</ol> 
<p><img alt="" class="alignnone wp-image-128233 size-full" height="513" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-14.png" width="1839" />Figure 14: Quick Topic suggested questions</p> 
<h2>Dashboard Build and Publish with Amazon Q</h2> 
<p>Amazon Q in Quick lets authors build dashboards using natural language — describe the visual you want, and Q generates it. This accelerates dashboard development from days to minutes and keeps the focus on business storytelling rather than chart configuration.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Analyses → Create analysis</strong>.</li> 
 <li>Select the <code>TPC-H Unified (Joined)</code>&nbsp;dataset.</li> 
 <li>Open the <a href="https://docs.aws.amazon.com/quick/latest/userguide/starting-from-questions-on-sheets.html" rel="noopener noreferrer" target="_blank"><strong>Amazon Q</strong> panel</a> .</li> 
 <li>Use natural language prompts to build each visual and <strong>Add to Analysis</strong>: 
  <ol type="a"> 
   <li><em>“</em>Show a KPI card for total revenue<em>.”</em></li> 
   <li><em>“Add a bar chart showing extended revenue by order status.”</em></li> 
   <li><em>“Create a scatter plot of discount rate versus extended revenue by customer segment.”</em></li> 
  </ol> </li> 
 <li>For each generated visual, review the field mappings and adjust titles, axis labels, and color encoding to match your organization’s style guide.</li> 
 <li>Add a <strong>filter control</strong> on <code>o_orderdate</code> so dashboard viewers can scope the data to a time range of their choice without requesting a new report.</li> 
 <li>Click <strong>Manage Q&amp;A </strong>to choose radio button and select <em><code>TPC-H Analytics</code></em><strong>topic </strong>for enabling Dashboard Q&amp;A. This embeds a natural language query bar directly in the published dashboard, allowing viewers to ask follow-up questions without leaving the dashboard. Quick automatically extracts semantic information from the dashboard visuals to power the Q&amp;A experience.</li> 
 <li>Select <strong>Publish</strong>, name it <code>TPC-H Lakehouse Analytics</code>.</li> 
 <li>Optionally, Quick allows to share dashboard.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-128234 size-full" height="460" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-15.png" width="1012" /></p> 
<p>Figure 15: Share Dashboard</p> 
<h2>Agentic AI Integration with Amazon Quick</h2> 
<p>Your SPICE datasets are loaded, your Topic is published, and your dashboard is live. Each of these is valuable on its own. Together, unified inside a Quick Space, surfaced through a custom Chat Agent and indexed Knowledge Base, they become something qualitatively different: an agentic AI system that answers questions, retrieves context, and drives action — all from a single conversational interface.</p> 
<h3>Knowledge Base Configuration</h3> 
<p>The Knowledge Base gives the Chat Agent access to unstructured context that structured data alone cannot answer — data dictionaries, schema documentation, business rules, and domain reference material. For this solution, the Knowledge Base is built from <strong>TPC-H unstructured data</strong>: the official TPC-H specification document describing how your organization maps TPC-H fields to business concepts.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Integrations → Knowledge bases → Webcrawler</strong>.</li> 
 <li>Add TPC-H specification (PDF) document content URL : <a href="https://www.tpc.org/tpc_documents_current_versions/pdf/tpc-h_v2.17.1.pdf" rel="noopener noreferrer" target="_blank">https://www.tpc.org/tpc_documents_current_versions/pdf/tpc-h_v2.17.1.pdf</a>.</li> 
 <li>Name the knowledge base <code>TPC-H Reference Knowledge Base</code>.</li> 
 <li>Select <strong>Create.</strong></li> 
</ol> 
<p>Quick indexes the document and makes it searchable by the Chat Agent at query time. The agent retrieves relevant passages — not entire document — so responses stay grounded and concise.</p> 
<p><strong>Best practice:</strong> Keep each document focused on a single topic. A 5-page data dictionary is more useful to the agent than a 200-page combined specification, because the agent retrieves by relevance — smaller, focused documents produce more precise retrievals.</p> 
<h3>Space Creation</h3> 
<p>A Quick Space is the organizational layer that abstracts your data assets — Topics, Knowledge Bases, dashboards, and datasets — into a single, governed context boundary. The Chat Agent you build in the next step does not query Topics and Knowledge Bases directly. It queries the Space. This design gives you one place to manage what the agent knows, who can access it, and what it is allowed to surface.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Spaces → Create space</strong>.</li> 
 <li>Name the space <code>TPC-H Lakehouse Analytics Space</code>.</li> 
 <li>Add resources to the space:</li> 
</ol> 
<p><strong>Add the Topic:</strong></p> 
<ul> 
 <li>Select <strong>Add knowledge → Topics</strong>.</li> 
 <li>Choose <code>TPC-H Analytics</code>&nbsp;(the Topic configured in the Quick Topic Configuration section).</li> 
 <li>The agent can now answer structured data questions — revenue, orders, customer segments — by querying the Topic through the Space.</li> 
</ul> 
<p><strong>Add the Knowledge Base:</strong></p> 
<ul> 
 <li>Select <strong>Add knowledge → Knowledge bases</strong>.</li> 
 <li>Choose <code>TPC-H Reference Knowledge Base</code>&nbsp;(the Knowledge Base configured in the Knowledge Base Configuration section).</li> 
 <li>The agent can now retrieve unstructured context from the TPC-H specification document — including the business intent of all 22 benchmark queries, query definitions, and the conceptual data model. When a user asks <em>“What is TPC-H Query 3 designed to measure?”</em> or <em>“What does the TPC-H specification say about order priority?”</em>, the agent retrieves the relevant passage from the specification and cites it in the response.</li> 
</ul> 
<p><strong>Add the Dashboard:</strong></p> 
<ul> 
 <li>Select <strong>Add knowledge → Dashboards</strong>.</li> 
 <li>Choose <code>TPC-H Lakehouse Analytics</code>&nbsp;(the dashboard configured in the Dashboard Build and Publish with Amazon Q section)</li> 
 <li>The agent can reference dashboard visuals and direct users to specific views when answering questions.</li> 
</ul> 
<p>The Space now encapsulates everything the Chat Agent needs: structured data through the Topic, unstructured context through the Knowledge Base, and visual references through the Dashboard. The agent queries the Space; the Space enforces the boundaries. Quick enforces the same security rules from the underlying knowledge inside the Space — users in the Space see only the data their role permits, regardless of how they ask the question.</p> 
<p><img alt="" class="alignnone wp-image-128235 size-full" height="521" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-16.png" width="1832" />Figure 16: Artifacts in Space</p> 
<h3>Custom Chat Agent Creation</h3> 
<p>The Chat Agent is the interface your business users interact with. It is not a generic assistant — it is a purpose-built, governed AI teammate scoped to <code>TPC-H Lakehouse Analytics Space</code>. Users ask questions in plain English. The agent reasons over the Space, retrieves the right combination of structured data and unstructured context, and returns grounded, cited answers.</p> 
<p><strong>Steps:</strong></p> 
<ol> 
 <li>In Amazon Quick, navigate to <strong>Chat agents → Create chat agent</strong>.</li> 
 <li>Write the <strong>persona instructions</strong> in plain language:</li> 
</ol> 
<p><em>“You are the TPC-H Analytics Agent for [Your Organization]. You help business analysts and data engineers answer questions about order revenue, supplier performance, line item pricing, and inventory availability using the TPC-H lakehouse dataset. Always ground your answers in data from the TPC-H Lakehouse Analytics Space. When a user asks a question that requires a chart or table, retrieve the answer from the Topic and present it clearly. When a user asks about schema definitions, query logic, or data dictionary terms, retrieve the answer from the Knowledge Base. Do not speculate. If you cannot find a grounded answer, say so and suggest a follow-up question.”</em></p> 
<ol start="3"> 
 <li>Enter a name: <code>TPC-H Analytics Agent</code>.</li> 
 <li><strong>Attach the Space: </strong>Quick can identify and attach <code>TPC-H Lakehouse Analytics Space</code><strong>. </strong>Optionally, you add the space with the following steps. 
  <ol type="a"> 
   <li>Under <strong>Knowledge sources</strong>, select <strong>Link spaces</strong>.</li> 
   <li>Choose <code>TPC-H Lakehouse Analytics Space</code>.</li> 
   <li>The agent now has access to the Topic, Knowledge Base, and Dashboard through the Space — no direct dataset connections are needed.</li> 
  </ol> </li> 
 <li><strong>Configure customization options:</strong> 
  <ol type="a"> 
   <li><strong>Welcome message</strong>: Add a custom greeting that appears when users first open the chat agent (e.g., <em>“Hello! I’m your TPC-H Analytics Agent. Ask me about order revenue, customer segments, or line item pricing.”</em>)</li> 
   <li><strong>Suggested prompts</strong>: Add 3-5 starter questions to guide users on what the agent can answer (e.g., <em>“What was total revenue last quarter?”</em>, <em>“Show me top customers by order volume”</em>, <em>“Explain the Shipping Priority Query”</em>)</li> 
   <li>These customization options help users understand the agent’s capabilities immediately and reduce the learning curve for first-time interactions.</li> 
  </ol> </li> 
 <li><strong>Preview and test the agent</strong> using the built-in preview panel on the right side of the configuration page before publishing. Test with questions that span both data sources: 
  <ol type="a"> 
   <li><em>“What was total revenue for fulfilled orders last quarter?”</em> — retrieves from the Topic and Dashboard (structured data).</li> 
   <li><em>“What does the l_shipmode field represent?”</em> — retrieves from the Knowledge Base (TPC-H specification).</li> 
   <li><em>“Show me the top 5 customer segments by order volume.”</em> — retrieves from the Topic and returns a ranked result.</li> 
   <li><em>“What business question does the Shipping Priority Query answer?”</em> — retrieves from Section 2.4.3 of the TPC-H specification in the Knowledge Base.</li> 
  </ol> </li> 
 <li>Select <strong>Launch chat agent</strong> to save and publish the changes.</li> 
</ol> 
<p><img alt="" class="alignnone wp-image-128236 size-full" height="717" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/ML-20574-image-17.png" width="1791" />Figure 17: Interact with Agent</p> 
<h3>What Your Users Experience</h3> 
<p>A business analyst opens the TPC-H Analytics Agent and types:</p> 
<p><em>“Which customer segment drove the most revenue last month, and what does ‘market segment’ mean in the TPC-H schema?”</em></p> 
<p>The agent:</p> 
<ol> 
 <li>Queries the <code>TPC-H Analytics</code> Topic through the Space for revenue by <code>c_mktsegment</code> filtered to last month — returning a ranked result from SPICE.</li> 
 <li>Simultaneously retrieves the definition of <code>c_mktsegment</code> from the TPC-H data dictionary in the Knowledge Base.</li> 
 <li>Returns a single, unified answer: the ranked revenue result with a citation to the SPICE dataset, followed by the schema definition with a citation to the specification document.</li> 
</ol> 
<p>No SQL. No dashboard navigation. No ticket to the data team. The answer arrives in one response, grounded in two sources, with every claim traceable to its origin.</p> 
<h2>Cleanup</h2> 
<p>Run following steps to remove the artifacts created by this blog post</p> 
<h3>Lakehouse / Data Lake Artifacts</h3> 
<p>Run following steps using Athena console</p> 
<h4>Drop Tables</h4> 
<pre><code class="language-sql">DROP TABLE blog_qs_athena_tpc_h_db_sql.customer_csv;
DROP TABLE blog_qs_athena_tpc_h_db_sql.orders_csv;
DROP TABLE blog_qs_athena_tpc_h_db_sql.orders_iceberg;
DROP TABLE blog_qs_athena_tpc_h_db_sql.lineitem_csv;
DROP TABLE lineitem_csv_s3_table; --(use S3 catalog configuration) </code></pre> 
<h4>Drop Databases</h4> 
<pre><code class="language-sql">DROP DATABASE blog_qs_athena_tpc_h_db_sql; </code></pre> 
<h4>Drop S3 Table bucket</h4> 
<ul> 
 <li>To delete <code>lineitem_csv_s3_table</code> table, use the AWS CLI, AWS SDKs, or Amazon S3 REST API. <a href="https://docs.aws.amazon.com/console/s3/tables-delete" rel="noopener noreferrer" target="_blank">Learn more</a></li> 
 <li>To delete namespace <code>blog_qs_athena_tpc_h_namespace</code>, use the AWS CLI, AWS SDKs, or Amazon S3 REST API. <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-namespace-delete.html" rel="noopener noreferrer" target="_blank">Learn more</a></li> 
 <li>To delete <code>blog-qs-athena-tpc-h-db-sql-s3-table-mar-3</code> table bucket, use the AWS CLI, AWS SDKs, or Amazon S3 REST API. <a href="https://docs.aws.amazon.com/console/s3/tables-bucket-delete" rel="noopener noreferrer" target="_blank">Learn more</a></li> 
</ul> 
<h4>Drop S3 bucket</h4> 
<p>Use S3 console to remove S3 bucket <code>amzn-s3-demo-bucket.</code></p> 
<h3>Quick Artifacts</h3> 
<h4>Delete the Custom Chat Agent</h4> 
<ul> 
 <li>In Amazon Quick, navigate to <strong>Agents</strong>.</li> 
 <li>Select <code>TPC-H Analytics Agent</code>&nbsp;and choose <strong>Delete</strong>.</li> 
 <li>Confirm the deletion.</li> 
</ul> 
<h4>Delete the Space</h4> 
<ul> 
 <li>Navigate to <strong>Spaces</strong>.</li> 
 <li>Select <code>TPC-H Lakehouse Analytics Space</code>&nbsp;and choose <strong>Delete</strong>.</li> 
 <li>Confirm the deletion. This removes the Space but does not delete the underlying Topics, Knowledge Bases, or Dashboards — those must be deleted separately.</li> 
</ul> 
<h4>Delete the Dashboard</h4> 
<ul> 
 <li>Navigate to <strong>Dashboards</strong>.</li> 
 <li>Select <code>TPC-H Lakehouse Analytics</code>&nbsp;and choose <strong>Delete</strong>.</li> 
 <li>Confirm the deletion.</li> 
</ul> 
<h4>Delete the Topic</h4> 
<ul> 
 <li>Navigate to <strong>Topics</strong>.</li> 
 <li>Select <code>TPC-H Analytics</code>&nbsp;and choose <strong>Delete</strong>.</li> 
 <li>Confirm the deletion.</li> 
</ul> 
<h4>Delete the Knowledge Base</h4> 
<ul> 
 <li>Navigate to <strong>Integrations → Knowledge bases</strong>.</li> 
 <li>Select <code>TPC-H Reference Knowledge Base</code>&nbsp;and choose <strong>Delete knowledge base</strong>.</li> 
 <li>Confirm the deletion. This removes the Knowledge Base and the indexed documents.</li> 
</ul> 
<h4>Delete the Datasets</h4> 
<ul> 
 <li>Navigate to <strong>Datasets</strong>.</li> 
 <li>Select each of the following datasets and choose <strong>Delete</strong>: 
  <ul> 
   <li><code>TPC-H Unified (Joined)</code></li> 
   <li><code>TPC-H Customer (CSV)</code></li> 
   <li><code>TPC-H Orders (Iceberg)</code></li> 
   <li><code>TPC-H Lineitem (S3 Tables)</code></li> 
  </ul> </li> 
 <li>Confirm each deletion. This removes the SPICE data and frees the associated SPICE capacity.</li> 
</ul> 
<h4>Delete the Data Source</h4> 
<ul> 
 <li>Navigate to <strong>Datasets → Data sources</strong>.</li> 
 <li>Select <code>tpch-lakehouse-athena</code>&nbsp;and choose <strong>Delete</strong>.</li> 
 <li>Confirm the deletion.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>This architecture demonstrates how Amazon Quick’s agentic AI transforms enterprise data analytics from a technical bottleneck into an accessible self-service capability. By integrating Amazon S3, AWS Glue Data Catalog, Amazon Athena, and Amazon Lake Formation with Amazon Quick’s conversational AI agents and dashboards, business users can now query complex lakehouse data through natural language interfaces without requiring SQL or BI expertise. The solution seamlessly combines structured TPC-H datasets across multiple storage formats (S3 Table, Iceberg, Parquet) with unstructured data from knowledge bases, enabling richer contextual insights. This democratization of data access accelerates decision-making across industries while maintaining enterprise-grade security, governance, and scalability for modern data-driven organizations.</p> 
<h2>Next steps</h2> 
<p>Reference <a href="https://docs.aws.amazon.com/quick/latest/userguide/getting-started-admin.html" rel="noopener" target="_blank">Getting started tutorial</a> for additional use cases using B2B, revenue, sales, marketing, and HR datasets. To dive deeper in Lake Formation permission with Quick reference AWS documentation “<a href="https://docs.aws.amazon.com/lake-formation/latest/dg/qs-integ-lf.html" rel="noopener noreferrer" target="_blank">Using AWS Lake Formation with Quick</a>“ and blog post – “<a href="https://aws.amazon.com/blogs/big-data/securely-analyze-your-data-with-aws-lake-formation-and-amazon-quicksight/" rel="noopener noreferrer" target="_blank">Securely analyze your data with AWS Lake Formation and Amazon Quick Sight”</a>. Join <a href="https://community.amazonquicksight.com/" rel="noopener noreferrer" target="_blank">Amazon Quick Community</a> to find answers to your questions, learning resources, and events in your area.</p> 
<p>For additional read reference following links –</p> 
<p><a href="https://aws.amazon.com/blogs/big-data/modernize-business-intelligence-workloads-using-amazon-quick/" rel="noopener noreferrer" target="_blank">Modernize Business Intelligence Workloads Using Amazon Quick</a></p> 
<p><a href="https://aws.amazon.com/blogs/business-intelligence/best-practices-for-amazon-quicksight-spice-and-direct-query-mode/" rel="noopener noreferrer" target="_blank">Best practices for Amazon Quick Sight SPICE and direct query mode</a></p> 
<p><a href="https://builder.aws.com/content/3B8KSSr0Z4DjwG3l2onzdGrnVU9/accessing-amazon-s3-tables-through-amazon-quick-with-aws-lake-formation-permissions" rel="noopener noreferrer" target="_blank">Accessing Amazon S3 Tables through Amazon Quick with AWS Lake Formation Permissions</a> <a href="https://docs.aws.amazon.com/quick/latest/userguide/security.html" rel="noopener noreferrer" target="_blank">AWS security in Quick</a>.</p> 
<hr style="width: 80%;" /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-128205" height="128" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/Screenshot-2026-04-09-at-12.39.47 PM.png" width="100" />
  </div> 
  <h3 class="lb-h4">Raj Balani</h3> 
  <p><a href="https://www.linkedin.com/in/rajbalani/" rel="noopener" target="_blank">Raj</a>&nbsp;is a Solutions Architect at Amazon Web Services. She enjoys exploring new cloud architectures and helping customers navigate their cloud journey with innovative solutions</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-128206" height="111" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/Screenshot-2026-04-09-at-12.44.36 PM.png" width="100" />
  </div> 
  <h3 class="lb-h4">Praney Mahajan</h3> 
  <p><a href="https://www.linkedin.com/in/pranaymahajan/" rel="noopener" target="_blank">Praney</a> is a Senior Technical Account Manager at AWS who partners with key enterprise customers as their strategic advisor. He is passionate about bridging technical solutions with business outcomes. He enjoys going on long drives with his family and playing cricket in his free time.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignnone wp-image-128207" height="114" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/09/My-Photo-262x300.png" width="100" />
  </div> 
  <h3 class="lb-h4">Rahul Sonawane</h3> 
  <p><a href="https://www.linkedin.com/in/rahul-sonawane-info" rel="noopener" target="_blank">Rahul</a> is a Principal Specialty Solutions Architect – GenAI/ML and Analytics at Amazon Web Services.</p> 
 </div> 
</footer>
