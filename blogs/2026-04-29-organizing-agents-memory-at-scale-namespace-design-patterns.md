---
title: "Organizing Agents’ memory at scale: Namespace design patterns in AgentCore Memory"
url: "https://aws.amazon.com/blogs/machine-learning/organizing-agents-memory-at-scale-namespace-design-patterns-in-agentcore-memory/"
date: "Wed, 29 Apr 2026 19:31:54 +0000"
author: "Noor Randhawa"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p>When building AI agents, developers struggle with organizing memory across sessions, which leads to irrelevant context retrieval and security vulnerabilities. AI agents that remember context across sessions need more than only storage. They need organized, retrievable, and secure memory. In Amazon Bedrock AgentCore Memory, <em>namespaces</em> determine how long-term memory records are organized, retrieved, and who can access them. Getting the namespace design right is essential to building an effective memory system.</p> 
<p>In this post, you will learn how to design namespace hierarchies, choose the right retrieval patterns, and implement AWS Identity and Access Management (IAM)-based access control for AgentCore Memory. If you’re new to AgentCore Memory, we recommend reading our introductory blog post first: <a href="https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-agentcore-memory-building-context-aware-agents/" rel="noopener noreferrer" target="_blank">Amazon Bedrock AgentCore Memory: Building context-aware agents</a>.</p> 
<h2>What are namespaces?</h2> 
<p><em>Namespaces</em> are hierarchical paths that organize long-term memory records within an AgentCore Memory resource. Think of them like directory paths in a file system. They provide logical structure, enable scoped retrieval, and support access control.</p> 
<p>When AgentCore Memory extracts long-term memory records from your conversations, each memory record is stored under a namespace. For example, a user’s preferences might live under <code>/actor/customer-123/preferences/</code>, while their session summaries might be stored at <code>/actor/customer-123/session/session-789/summary/</code>. With this structure, you can retrieve memory records at exactly the right level of granularity.</p> 
<p>If you’ve worked with partition keys in Amazon DynamoDB or folder structures in Amazon Simple Storage Service (Amazon S3), the mental model transfers well. Just as you think through access patterns before choosing a partition key or designing your S3 folder hierarchy, you should think through your retrieval patterns before designing your namespace structure. Determine:</p> 
<ul> 
 <li>Who needs to access these memories: A single user? All users of an agent?</li> 
 <li>Granularity of retrieval you need: Is it per-session summaries? Cross-session preferences?</li> 
 <li>Isolation boundaries that matter: Should one user’s memories ever be visible to another? Agent-scoped memories?</li> 
</ul> 
<p>The main difference from a partition key is that namespaces support hierarchical retrieval in addition to exact match. You can query at each level of the hierarchy, not only at the leaf level. You can use a well-designed namespace to retrieve memories scoped to a single session, a single user across sessions, or a broader grouping, from the same memory resource. Namespaces are logical groupings within the same underlying storage. They provide organizational structure and access control, but long-term memory records across different namespaces co-exist within the same memory resource. Your hierarchy is your primary tool for organizing data for effective retrieval patterns.</p> 
<h2>Namespace templates and resolution</h2> 
<p>When creating a memory resource, you define namespace templates using the <code>namespaceTemplate</code> field within each strategy configuration. Templates support three pre-defined variables:</p> 
<ul> 
 <li><code>{actorId}</code> – resolves to the actor identifier from the events being processed</li> 
 <li><code>{sessionId}</code> – resolves to the session identifier from the events</li> 
 <li><code>{memoryStrategyId}</code> – resolves to the strategy identifier</li> 
</ul> 
<p>Here’s an example of creating a memory resource with namespace templates:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">response = agentcore_client.create_memory(
&nbsp;&nbsp; &nbsp;name="CustomerSupportMemory",
&nbsp;&nbsp; &nbsp;description="Memory for customer support agents",
&nbsp;&nbsp; &nbsp;eventExpiryDuration=30,
&nbsp;&nbsp; &nbsp;memoryStrategies=[
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"semanticMemoryStrategy": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"name": "customer-facts",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"namespaceTemplate": "/actor/{actorId}/facts/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;},
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"summaryMemoryStrategy": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"name": "session-summaries",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"namespaceTemplate": "/actor/{actorId}/session/{sessionId}/summary/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;]
)</code></pre> 
</div> 
<p>When events arrive for <code>actorId=customer-456</code> in <code>sessionId=session-789</code>, the resolved namespaces become:</p> 
<ul> 
 <li><code>/actor/customer-456/facts/</code></li> 
 <li><code>/actor/customer-456/session/session-789/summary/</code></li> 
</ul> 
<h2>Namespace design per memory strategy</h2> 
<p>Each memory strategy has different scoping needs, and the namespace design should reflect how that data will be accessed. Below are some common namespace design patterns for different memory strategies.</p> 
<h3>1. Semantic and user preferences: Actor-scoped</h3> 
<p>Semantic memory captures facts and knowledge from conversations (for example, “The customer’s company has 500 employees”). User Preference Memory captures choices and styles (for example, “User prefers Python for development work”). Both memory types accumulate over time and are relevant across sessions. A fact learned in January should still be retrievable in March. For these strategies, scope the namespace to the actor:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">/actor/{actorId}/facts/
/actor/{actorId}/preferences/</code></pre> 
</div> 
<p>This means the facts and preferences for a given user are consolidated under a single namespace, regardless of which session they were extracted from. The consolidation engine merges related memories within the same namespace. Look at figure 1 for an example of how scoping impacts the consolidation logic.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/ML-20833-image-1.png"><img alt="" class="alignleft size-full wp-image-129496" height="1731" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/ML-20833-image-1.png" width="3328" /></a></p> 
<p>The following diagram illustrates how actor-scoped semantic and preference memories are organized:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">Memory Resource: CustomerSupportMemory
│
├── /actor/customer-123/
│ &nbsp; ├── facts/
│ &nbsp; │ &nbsp; ├── "Company has 500 employees across Seattle, Austin, Boston"
│ &nbsp; │ &nbsp; ├── "Currently migrating from on-premises to cloud"
│ &nbsp; │ &nbsp; └── "Primary contact is the VP of Engineering"
│ &nbsp; └── preferences/
│ &nbsp; &nbsp; &nbsp; ├── "Prefers email communication over phone"
│ &nbsp; &nbsp; &nbsp; └── "Usually prefers detailed technical explanations"
│
├── /actor/customer-456/
│ &nbsp; ├── facts/
│ &nbsp; │ &nbsp; ├── "Startup with 20 employees"
│ &nbsp; │ &nbsp; └── "Using serverless architecture"
│ &nbsp; └── preferences/
│ &nbsp; &nbsp; &nbsp; └── "Prefers concise, high-level summaries"</code></pre> 
</div> 
<p>In some use cases, an admin might need to retrieve information across actors while keeping memories organized per actor. For example, a customer support agent might need to look up known issues reported by other customers, or a sales agent might need to find similar customer profiles across its user base. For such cases, structure the namespace with the actor identifier as a child of the memory type rather than as the parent:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">/customer-issues/{actorId}/
/sales/{actorId}/</code></pre> 
</div> 
<p>With this inverted structure, you can use <code>namespacePath="/customer-issues/"</code> to retrieve common issues raised across all customers, while still maintaining a per-actor organization. A query scoped to <code>namespace="/customer-issues/customer-123/"</code> returns only that actor’s reported issues, preserving isolation when needed.</p> 
<h3>2. Summary: Session-scoped</h3> 
<p>Summary memory creates running narratives of conversations, capturing main points and decisions. Instead of feeding an entire conversation history into the large language model’s (LLM) context window, you can retrieve a compact summary that preserves the key information while significantly reducing token usage. Because summaries are inherently tied to a specific conversation, they should include the session identifier:<code>/actor/{actorId}/session/{sessionId}/summary/</code>This scoping means that each session gets its own summary, while still being organized under the actor for cross-session retrieval when needed.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">Memory Resource: CustomerSupportMemory
│
├── /actor/customer-123/
│ &nbsp; ├── session/session-001/summary/
│ &nbsp; │ &nbsp; └── "Customer inquired about enterprise pricing, discussed
│ &nbsp; │ &nbsp; &nbsp; &nbsp; &nbsp;implementation timeline, requested follow-up demo"
│ &nbsp; ├── session/session-002/summary/
│ &nbsp; │ &nbsp; └── "Follow-up on demo scheduling, confirmed Q3 timeline,
│ &nbsp; │ &nbsp; &nbsp; &nbsp; &nbsp;discussed integration requirements with existing CRM"
│ &nbsp; └── session/session-003/summary/
│ &nbsp; &nbsp; &nbsp; └── "Technical deep-dive on API integration, reviewed
│ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;authentication options, chose OAuth 2.0 approach"</code></pre> 
</div> 
<h3>3. Episodic: Session-scoped with reflection hierarchy</h3> 
<p>Episodic memory captures complete reasoning traces, including the goal, steps taken, outcomes, and reflections. Because each episode represents what happened during a specific interaction, episodes should be scoped to the session, similar to summaries. For example, a flight booking agent might store an episode capturing how it searched for flights, compared options, handled a fare class restriction, and ultimately rebooked the customer on an alternative route. That episode belongs to the session where it occurred. Reflections are cross-episode insights stored at a parent level. They generalize learnings across sessions, for instance “when a fare class restriction blocks a modification, immediately search for alternative flights rather than just explaining the policy.” The namespace for reflections must be a sub-path of the namespace for episodes:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">Episodes: &nbsp; &nbsp;/actor/{actorId}/session/{sessionId}/episodes/
Reflections: /actor/{actorId}/</code></pre> 
</div> 
<h2><strong>Retrieval patterns</strong></h2> 
<h3>Retrieval APIs</h3> 
<p>AgentCore Memory provides three primary retrieval APIs for long-term memory, each suited to different access patterns. Choosing the right one is key to building effective agents.</p> 
<h4>1. Semantic search with RetrieveMemoryRecords</h4> 
<p>Use <code>RetrieveMemoryRecords</code> to find memories that are semantically relevant to a query. This is the primary retrieval method during agent interactions, surfacing the most relevant memories based on meaning, rather than exact text matching.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># Retrieve memories relevant to the current user query
memories = agentcore_client.retrieve_memory_records(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;namespace="/actor/customer-123/facts/",
&nbsp;&nbsp; &nbsp;searchCriteria={
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"searchQuery": "What cloud migration approach is the customer using?",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"topK": 5
&nbsp;&nbsp; &nbsp;}
)</code></pre> 
</div> 
<p>The search query can come from two sources:</p> 
<ul> 
 <li><strong>Directly from the user query –</strong> Pass the user’s question as-is when it naturally maps to the kind of information stored in memory. For example, if the user asks “What’s my budget?”, that query works well for retrieving preference or fact memories.</li> 
 <li><strong>LLM-generated query –</strong> For more complex scenarios, have your agent’s LLM formulate a targeted search query. This is useful when the user’s raw input doesn’t directly map to stored memories. For example, if the user says “Help me plan my next trip,” the LLM might generate a search query like “travel preferences, destination history, budget constraints” to retrieve the most relevant memories. Note that this adds latency.</li> 
</ul> 
<h4>2. Direct retrieval with ListMemoryRecords</h4> 
<p>Use <code>ListMemoryRecords</code> when you need to enumerate memories within a specific namespace such as, displaying a user’s stored preferences in a console UI, auditing what memories exist, or performing bulk operations.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code"># List all memories in a specific namespace
records = agentcore_client.list_memory_records(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;namespace="/actor/customer-123/preferences/"
)</code></pre> 
</div> 
<h4>3. GetMemoryRecord and DeleteMemoryRecord</h4> 
<p>When you know the specific memory record ID (for example, from a previous list or retrieve call), use <code>GetMemoryRecord</code> for direct lookup or <code>DeleteMemoryRecord</code> to remove a specific memory:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code"># Get a specific memory record
record = agentcore_client.get_memory_record(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;memoryRecordId="rec-abc123"
)

# Delete a specific memory record
agentcore_client.delete_memory_record(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;memoryRecordId="rec-abc123"
)</code></pre> 
</div> 
<p>These are useful for memory management workflows that are used to help users view, correct, or delete specific memories through your application’s UI.</p> 
<h3><strong>Namespace vs. NamespacePath: Exact match vs. hierarchical retrieval</strong></h3> 
<p>AgentCore Memory provides two distinct fields for scoping retrieval, and understanding the difference is critical for correct behavior.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/ML-20833-image-2.png"><img alt="" class="alignleft size-full wp-image-129497" height="1031" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/ML-20833-image-2.png" width="3302" /></a></p> 
<h4>1. namespace — Exact match</h4> 
<p>The <code>namespace</code> field performs an <strong>exact match</strong>. It returns only memory records stored at that precise namespace path.</p> 
<div class="hide-language"> 
 <pre><code class="lang-css"># Returns ONLY records stored at /actor/customer-123/facts/
records = agentcore_client.retrieve_memory_records(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;namespace="/actor/customer-123/facts/",
&nbsp;&nbsp; &nbsp;searchCriteria={
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"searchQuery": "cloud migration",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"topK": 5
&nbsp;&nbsp; &nbsp;}
)</code></pre> 
</div> 
<p>This is the right choice when you know exactly which namespace you want to query and need precise scoping. For example, retrieving only a user’s preferences without pulling in their facts or summaries.</p> 
<h4>2. namespacePath — Hierarchical retrieval</h4> 
<p>The <code>namespacePath</code> field performs a <strong>hierarchical match</strong>, returning the memory records whose namespace falls under the specified path.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># Returns records from 
# /actor/customer-123/facts/,
# /actor/customer-123/preferences/,
# /actor/customer-123/session/*/summary/, etc.
records = agentcore_client.retrieve_memory_records(
&nbsp;&nbsp; &nbsp;memoryId="mem-12345abcdef",
&nbsp;&nbsp; &nbsp;namespacePath="/actor/customer-123/",
&nbsp;&nbsp; &nbsp;searchCriteria={
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"searchQuery": "cloud migration",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"topK": 5
&nbsp;&nbsp; &nbsp;}
)</code></pre> 
</div> 
<p>This is useful when you want to search across the user’s memories regardless of type, or when building features like “show me everything we know about this customer.” Note that it’s important that you think through your isolation and retrieval patterns to make sure that tree traversal doesn’t expose unintended data.</p> 
<h3>When to use which</h3> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px;"></td> 
   <td style="padding: 10px;">Scenario</td> 
   <td style="padding: 10px;">API</td> 
   <td style="padding: 10px;">Field</td> 
   <td style="padding: 10px;">Example</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">1</td> 
   <td style="padding: 10px;">Retrieve semantically relevant user preferences</td> 
   <td style="padding: 10px;">RetrieveMemoryRecords</td> 
   <td style="padding: 10px;">namespace</td> 
   <td style="padding: 10px;">/actor/customer-123/preferences/</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">2</td> 
   <td style="padding: 10px;">Retrieve a specific session summary</td> 
   <td style="padding: 10px;">ListMemoryRecords</td> 
   <td style="padding: 10px;">namespace</td> 
   <td style="padding: 10px;">/actor/customer-123/session/session-001/summary/</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">3</td> 
   <td style="padding: 10px;">List all preferences for a user</td> 
   <td style="padding: 10px;">ListMemoryRecords</td> 
   <td style="padding: 10px;">namespace</td> 
   <td style="padding: 10px;">/actor/customer-123/preferences/</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">4</td> 
   <td style="padding: 10px;">Search across a user’s memories</td> 
   <td style="padding: 10px;">RetrieveMemoryRecords</td> 
   <td style="padding: 10px;">namespacePath</td> 
   <td style="padding: 10px;">/actor/customer-123/</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">5</td> 
   <td style="padding: 10px;">List summaries across sessions for a user</td> 
   <td style="padding: 10px;">ListMemoryRecords</td> 
   <td style="padding: 10px;">namespacePath</td> 
   <td style="padding: 10px;">/actor/customer-123/session/</td> 
  </tr> 
 </tbody> 
</table> 
<h3>Writing IAM policies for namespace access control</h3> 
<p>Namespaces integrate with AWS Identity and Access Management (IAM) through condition keys that restrict which namespaces a principal can include in their Memory API requests.</p> 
<h4>1. Exact match policies</h4> 
<p>Use <code>StringEquals</code> with the <code>bedrock-agentcore:namespace</code> condition key to restrict access to a specific namespace:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
&nbsp;&nbsp;"Version": "2012-10-17",
&nbsp;&nbsp;"Statement": [
&nbsp;&nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:RetrieveMemoryRecords",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:ListMemoryRecords"
&nbsp;&nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp;"Resource": "arn:aws:bedrock-agentcore:us-east-1:123456789012:memory/mem-12345abcdef",
&nbsp;&nbsp; &nbsp; &nbsp;"Condition": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"StringEquals": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:namespace": "/actor/${aws:PrincipalTag/userId}/preferences/"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;}
&nbsp;&nbsp;]
}</code></pre> 
</div> 
<p>This policy makes sure that a user can only retrieve memories from their own preferences namespace, using the <code>userId</code> principal tag (injected) for dynamic scoping.</p> 
<h4>2. Hierarchical retrieval policies</h4> 
<p>Use <code>StringLike</code> with the <code>bedrock-agentcore:namespacePath</code> condition key for hierarchical access:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
&nbsp;&nbsp;"Version": "2012-10-17",
&nbsp;&nbsp;"Statement": [
&nbsp;&nbsp; &nbsp;{
&nbsp;&nbsp; &nbsp; &nbsp;"Effect": "Allow",
&nbsp;&nbsp; &nbsp; &nbsp;"Action": [
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:RetrieveMemoryRecords",
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:ListMemoryRecords"
&nbsp;&nbsp; &nbsp; &nbsp;],
&nbsp;&nbsp; &nbsp; &nbsp;"Resource": "arn:aws:bedrock-agentcore:us-east-1:123456789012:memory/mem-12345abcdef",
&nbsp;&nbsp; &nbsp; &nbsp;"Condition": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;"StringLike": {
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;"bedrock-agentcore:namespacePath": "/actor/${aws:PrincipalTag/userId}/*"
&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp; &nbsp;}
&nbsp;&nbsp; &nbsp;}
&nbsp;&nbsp;]
}</code></pre> 
</div> 
<p>With this, a user can perform hierarchical retrieval across their namespaces (facts, preferences, summaries) while helping prevent access to other users’ data.</p> 
<h2>Conclusion</h2> 
<p>Namespace design is foundational to building effective memory systems with AgentCore Memory. Much like designing a key schema for a database or a prefix structure in object storage, thinking through your access patterns upfront helps you create a namespace hierarchy that supports precise retrieval, clean isolation between users, and IAM-based access control.The key takeaways:</p> 
<ul> 
 <li><strong>Think through your access patterns and isolation boundaries</strong> before coming up with namespace templates</li> 
 <li><strong>Scope semantic and preference memories to the actor</strong> (<code>/actor/{actorId}/</code>) for cross-session consolidation</li> 
 <li><strong>Scope summaries to the session</strong> (<code>/actor/{actorId}/session/{sessionId}/</code>) since they’re conversation-specific (where needed such as summaries or episodes)</li> 
 <li><strong>Use </strong><code>namespace</code><strong> for exact match</strong> when you know the precise path, and <code>namespacePath</code><strong> for hierarchical retrieval</strong> when you need to search across a subtree</li> 
 <li><strong>Use leading and trailing slashes</strong> in namespace paths to keep them consistent and help prevent prefix collisions</li> 
 <li><strong>Use IAM condition keys</strong> (<code>bedrock-agentcore:namespace</code> and <code>bedrock-agentcore:namespacePath</code>) to control what namespaces can be requested</li> 
</ul> 
<p>To get started, visit the following resources:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/memory-organization.html" rel="noopener noreferrer" target="_blank">AgentCore Memory Organization Documentation</a></li> 
 <li><a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/specify-long-term-memory-organization.html">Agentcore Memory: Long term memory organization documentation</a></li> 
 <li><a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/04-AgentCore-memory" rel="noopener noreferrer" target="_blank">AWS Samples GitHub Repository</a></li> 
</ul> 
<hr style="width: 80%;" /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-thumbnail wp-image-129499" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/noorullr-100x133.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Noor Randhawa</h3> 
  <p><a href="https://www.linkedin.com/in/noorra/" rel="noopener" target="_blank">Noor Randhawa</a> is the Tech Lead for AgentCore Memory at Amazon Web Services (AWS), building systems that enable developers to create intelligent, context-aware agents powered by Memory. He previously worked across Amazon Retail and AWS EKS, designing highly scalable and distributed platforms.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft size-thumbnail wp-image-129500" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/akshseh_1-100x133.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Akarsha Sehwag</h3> 
  <p><a href="https://www.linkedin.com/in/akarshasehwag/" rel="noopener" target="_blank">Akarsha Sehwag</a> is a Generative AI Data Scientist with AgentCore Memory team. With over seven years of experience in AI/ML product development, she has delivered enterprise-grade solutions for customers across a wide range of industries. Outside of work, she enjoys learning new things and exploring the outdoors.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="alignleft wp-image-129503 size-thumbnail" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/24/piradeep-100x133.jpg" width="100" />
  </div> 
  <h3 class="lb-h4">Piradeep Kandasamy</h3> 
  <p><a href="https://www.linkedin.com/in/piradeep-k-39081241/" rel="noopener" target="_blank">Piradeep Kandasamy</a> is a Software Development Manager for AgentCore Memory. Over his career at Amazon, he has built and scaled systems across Amazon Alexa, Amazon ECS, and AWS CloudFormation, bringing deep expertise in distributed systems and large-scale cloud services to his current work on memory infrastructure for AI agents.</p> 
 </div> 
</footer>
