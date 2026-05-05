---
title: "Configuring Amazon Bedrock AgentCore Gateway for secure access to private resources"
url: "https://aws.amazon.com/blogs/machine-learning/configuring-amazon-bedrock-agentcore-gateway-for-secure-access-to-private-resources/"
date: "Thu, 30 Apr 2026 16:32:43 +0000"
author: "Eashan Kaushik"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
<p>AI agents in production environments often need to reach internal APIs, databases, and private resources that sit behind <a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud (Amazon VPC)</a> boundaries. Managing private connectivity for each agent-to-tool path adds operational overhead and slows deployment. Amazon Bedrock AgentCore <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/usingVPC.html" rel="noopener noreferrer" target="_blank">VPC connectivity</a> is designed to deploy AI agents and <a href="https://modelcontextprotocol.io/docs/getting-started/intro" rel="noopener noreferrer" target="_blank">Model Context Protocol (MCP)</a> servers without requiring the network traffic to be exposed to the public internet. This capability extends to managed Amazon VPC egress for <a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway.html" rel="noopener noreferrer" target="_blank">Amazon Bedrock AgentCore Gateway</a>, so you can connect to endpoints inside private networks across your AWS environment.</p> 
<p>In this post, you will configure Amazon Bedrock AgentCore Gateway to access private endpoints using<a href="https://docs.aws.amazon.com/vpc/latest/privatelink/resource-gateway.html" rel="noopener noreferrer" target="_blank"> Resource Gateway</a>, a managed construct that provisions <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html" rel="noopener noreferrer" target="_blank">Elastic Network Interfaces (ENIs)</a> directly inside your Amazon VPC, one per subnet. You will explore two implementation modes (managed and self-managed) and walk through three practical scenarios: connecting to a private <a href="https://aws.amazon.com/api-gateway/" rel="noopener noreferrer" target="_blank">Amazon API Gateway</a> endpoint, integrating with a MCP server on <a href="https://aws.amazon.com/eks/" rel="noopener noreferrer" target="_blank">Amazon Elastic Kubernetes Service</a> (Amazon EKS), and accessing a private REST API.</p> 
<h2>Key terms</h2> 
<p>The following terms are used throughout this post. Review them before proceeding to understand how each component fits into the AgentCore Gateway VPC egress architecture.</p> 
<p><strong>Resource VPC: </strong>The Amazon VPC where your private resource lives. For example, the VPC containing your privately hosted MCP server or API endpoint. This is the Amazon VPC that AgentCore Gateway needs to reach. Resource VPC can either be in the same AWS account as the AgentCore Gateway account or in a different account.</p> 
<p><strong>AgentCore Gateway account: </strong>The AWS account where you create and manage your AgentCore Gateway resources. This account may or may not be the same account as the Resource VPC.</p> 
<p><strong>Resource Gateway:</strong> <a href="https://docs.aws.amazon.com/vpc/latest/privatelink/resource-gateway.html" rel="noopener noreferrer" target="_blank">Resource gateway</a> acts as the private entry point into your Resource VPC. When created, it provisions one ENI per subnet that you specify, each sitting inside your VPC. Traffic from AgentCore Gateway to your private resource arrives through these ENIs.</p> 
<p><strong>Resource Configuration: </strong><a href="https://docs.aws.amazon.com/vpc/latest/privatelink/resource-configuration.html" rel="noopener noreferrer" target="_blank">Resource configuration for VPC resources</a> defines the specific resource AgentCore Gateway is allowed to reach through the Resource Gateway, identified by a domain name, or IP address. Rather than granting access to your entire Amazon VPC, a Resource Configuration scopes connectivity to a single endpoint.</p> 
<p><strong>Service Network Resource Association</strong>: A service network resource association connects a resource configuration to the AgentCore service network, which allows AgentCore Gateway service to invoke your private endpoint. AgentCore creates and manages this association on your behalf, regardless of which mode you use.</p> 
<h2>How does AgentCore Gateway VPC egress work?</h2> 
<p>AgentCore Gateway VPC egress supports two modes depending on how much control you want over the underlying networking infrastructure and how you want to architect for cross-VPC connectivity.</p> 
<h3>Managed VPC resource</h3> 
<p>In this mode, AgentCore Gateway handles everything on your behalf. You provide your VPC ID, subnet IDs, and security groups as part of your target configuration, and AgentCore automatically creates and manages the VPC Resource Gateway in your account. This mode integrates with existing network architectures, whether you use<a href="https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html" rel="noopener noreferrer" target="_blank"> VPC peering</a> for same-Region or cross-Region connectivity or a hub-and-spoke model with <a href="https://aws.amazon.com/transit-gateway/" rel="noopener noreferrer" target="_blank">AWS Transit Gateway</a> for multi-VPC and hybrid environments.</p> 
<p>The following architecture shows how AgentCore Gateway connects to private <a href="https://aws.amazon.com/api-gateway/" rel="noopener noreferrer" target="_blank">Amazon API Gateway</a> using managed VPC resource mode.</p> 
<p><img alt="" class="alignnone size-full wp-image-129870" height="516" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-1-12.png" width="1650" /></p> 
<p>When you create an AgentCore Gateway Target with a managed VPC resource configuration, AgentCore Gateway initiates the request and routes it to the Resource Gateway inside your Resource Owner VPC. The Resource Gateway forwards traffic through an ENI provisioned in your subnet, governed by the security groups you configure. From the ENI, the request reaches the <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html" rel="noopener noreferrer" target="_blank">execute-api VPC endpoint</a>. In managed VPC Resource mode, AgentCore creates and manages the Resource Gateway on your behalf, you only have read-only visibility into it.</p> 
<h3>Self-managed Lattice resource</h3> 
<p>In this mode, you create and manage the VPC Lattice Resource Gateway and Resource configuration before referencing it during target creation on AgentCore Gateway. This gives you visibility and control over the Resource configuration, including the number of IPv4 addresses per ENI, subnet placement, and security group rules. More importantly, it gives you visibility into the resource configuration itself, with the ability to view it, share it using <a href="https://aws.amazon.com/ram/" rel="noopener noreferrer" target="_blank">AWS Resource Access Manager (AWS RAM)</a> (required for cross-account connectivity), see associations tied to it, and retain the ability to revoke those associations when you choose.</p> 
<p>The following architecture shows how AgentCore Gateway connects to private Rest API endpoints using self-managed lattice resource mode.</p> 
<p><img alt="" class="alignnone size-full wp-image-129871" height="1274" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-2-9.png" width="3608" /></p> 
<p>In self-managed lattice resource mode, you pre-create the Resource Gateway and Resource Configuration before configuring your AgentCore Gateway Target. When you call <a href="https://docs.aws.amazon.com/bedrock-agentcore-control/latest/APIReference/API_CreateGatewayTarget.html" rel="noopener noreferrer" target="_blank">CreateGatewayTarget</a>, you pass the Resource Configuration ID to associate AgentCore Gateway target with your private endpoint. At invocation time, Resource Gateway forwards the request through an ENI provisioned in your subnet, governed by the security groups you configure. From the ENI, the request reaches the execute-api VPC endpoint. Unlike managed VPC Resource mode, you own and manage the Resource Gateway and Resource Configuration.</p> 
<p>Use the following table to determine which mode fits your architecture. Choose managed VPC resource mode for streamlined setup, or self-managed Lattice resource mode for control over Resource Gateway lifecycle, cross-account connectivity, and visibility into associations.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px;"><strong>Dimension </strong></td> 
   <td style="padding: 10px;"><strong>AgentCore Managed VPC Resource </strong></td> 
   <td style="padding: 10px;"><strong>Self-Managed Lattice Resource</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Setup complexity</td> 
   <td style="padding: 10px;">Straightforward; provide VPC ID, subnet IDs, and security group IDs. AgentCore manages the rest</td> 
   <td style="padding: 10px;">Advanced; you create and manage the Amazon VPC Lattice Resource Gateway and Resource Configurations yourself, then pass the resource configuration ID to each target</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">IPv4 consumption</td> 
   <td style="padding: 10px;">Each managed resource gateway consumes 1 IP address per ENI. This is not configurable</td> 
   <td style="padding: 10px;">When used with Amazon Bedrock AgentCore, it consumes one IP address per subnet. If also attached to other VPC Lattice service networks, it consumes additional IPs based on the ipv4AddressesPerEni value on the resource gateway</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Cross-account</td> 
   <td style="padding: 10px;">Not natively supported use hub-and-spoke architectures (<a href="https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html" rel="noopener noreferrer" target="_blank">VPC peering </a>or <a href="https://aws.amazon.com/transit-gateway/" rel="noopener noreferrer" target="_blank">AWS Transit Gateway</a> ) for cross-account / cross-VPC scenarios.</td> 
   <td style="padding: 10px;">Supported with <a href="https://aws.amazon.com/ram/" rel="noopener noreferrer" target="_blank">AWS Resource Access Manage (AWS RAM)</a>. Enables direct cross-account connectivity without requiring VPC peering or Transit Gateways.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Reusing existing ENIs</td> 
   <td style="padding: 10px;">AgentCore automatically reuses one Resource Gateway (and its ENIs) across targets in the account whose managedVpcResource config matches on Amazon VPC, subnet set, security-group set, tags, and IP address type</td> 
   <td style="padding: 10px;">You attach multiple Resource Configurations to a single Resource Gateway you own; target whose resourceConfigurationIdentifier resolves to that Resource Gateway shares its ENIs</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Resource Gateway Lifecycle management</td> 
   <td style="padding: 10px;">Amazon Bedrock AgentCore creates, reuses, and deletes Resource Gateways on your behalf</td> 
   <td style="padding: 10px;">You own the full lifecycle of resource gateways and resource configurations</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Governance and visibility</td> 
   <td style="padding: 10px;">Resource Configurations are managed in the Amazon Bedrock AgentCore service account and aren’t visible in your Amazon VPC console. The underlying Resource Gateway is visible in your account in read-only mode</td> 
   <td style="padding: 10px;">Full visibility into Resource Configurations, Service Network associations, and connected domains in your Amazon VPC Lattice console. You can audit connections and revoke access at a granular level</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px;">Pricing</td> 
   <td style="padding: 10px;">Per-GB data processing charges only (for data processed through the Resource Gateway)</td> 
   <td style="padding: 10px;">1) Hourly charge for Service Network association2) Per-GB data processing charges</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Get started with AgentCore Gateway VPC egress</h2> 
<p>In this post, you focus on the managed VPC resource mode. If you want to explore the self-managed lattice resource offering, follow the <a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/02-AgentCore-gateway/16-vpc-egress" rel="noopener noreferrer" target="_blank">code samples</a>. Before getting started, this post assumes basic familiarity with Amazon VPC, <a href="https://aws.amazon.com/cli/" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI), Amazon Bedrock AgentCore, and Amazon Bedrock AgentCore Gateway. Make sure that you have the following in place.</p> 
<ul> 
 <li><a href="https://aws.amazon.com/cli/" rel="noopener noreferrer" target="_blank">AWS Command Line Interface </a>(AWS CLI)</li> 
 <li><a href="https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html" rel="noopener noreferrer" target="_blank">AWS Credentials</a></li> 
 <li>Public Certificate Authority</li> 
</ul> 
<p>Currently AgentCore Gateway’s trusts publicly signed TLS certificates; it doesn’t trust certificates signed by a private CA, so the handshake to your backend fails. If your endpoint is protected by a private or self-signed certificate, find the working solution sample on <a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/02-AgentCore-gateway/16-vpc-egress" rel="noopener noreferrer" target="_blank">GitHub</a>.</p> 
<ul> 
 <li><a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> permissions</li> 
</ul> 
<p>Your IAM principal needs the iam:CreateServiceLinkedRole permission for bedrock-agentcore.amazonaws.com, so that AgentCore can create the service-linked role on your behalf if it does not already exist. For the required IAM policy, see Gateway <a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/service-linked-roles.html#gateway-service-linked-role" rel="noopener noreferrer" target="_blank">service-linked role</a>.</p> 
<ul> 
 <li>Set up security groups</li> 
</ul> 
<p>The Resource Gateway security group controls what traffic the Resource Gateway ENIs can send outbound to resources inside your Amazon VPC. If you don’t provide the security group while invoking <a href="https://docs.aws.amazon.com/bedrock-agentcore-control/latest/APIReference/API_CreateGatewayTarget.html" rel="noopener noreferrer" target="_blank">CreateGatewayTarget </a>API, then the default security group is used.</p> 
<ul> 
 <li>AgentCore Gateway</li> 
</ul> 
<p>This walkthrough assumes that you have an existing AgentCore Gateway. If you haven’t created one yet, run:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws bedrock-agentcore create-gateway \
  --name my-gateway \
  --role-arn arn:aws:iam::&lt;account-=id&gt;:role/AgentCoreGatewayRole</code></pre> 
</div> 
<p>Note the <code>gatewayId</code> from the response. You need it when creating AgentCore Gateway Targets in the steps that follow.</p> 
<ul> 
 <li>For in-depth examples, see the GitHub <a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/02-AgentCore-gateway/16-vpc-egress" rel="noopener noreferrer" target="_blank">repository </a>.</li> 
</ul> 
<h3>Private Amazon API Gateway</h3> 
<p>In this section, you create an AgentCore Gateway target that routes to a private Amazon API Gateway. Call the <a href="https://docs.aws.amazon.com/bedrock-agentcore-control/latest/APIReference/API_CreateGatewayTarget.html" rel="noopener noreferrer" target="_blank">CreateGatewayTarget </a>API with the following parameters. In the openApiSchema field, provide your private Amazon API Gateway endpoint URL (<code>https://{api-id}-{vpce-id}.execute-api.{region}.amazonaws.com/{stage}</code>). In the managedVpcResource block, provide your VPC ID, subnet IDs, and security group ID.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-clike">  aws bedrock-agentcore-control create-gateway-target \
    --region us-west-2 \
    --cli-input-json '{
      "gatewayIdentifier": "&lt;GATEWAY_ID&gt;",           
      "name": "private-apigw",
      "description": "Private API Gateway",                 
      "targetConfiguration": {                     
        "mcp": {
          "openApiSchema": {                     
            "inlinePayload": "..."
          }
        }
      },
      "credentialProviderConfigurations": [...],          
      "privateEndpoint": {
        "managedVpcResource": {
          "vpcIdentifier": "&lt;VPC_ID&gt;",
          "subnetIds": ["&lt;SUBNET_ID_1&gt;", "&lt;SUBNET_ID_2&gt;"],
          "endpointIpAddressType": "IPV4",
          "securityGroupIds": ["&lt;VPCE_SG_ID&gt;"]
        }                                        
      }           
    }'       
</code></pre> 
</div> 
<p>After you run the command, AgentCore Gateway uses its service-linked role to provision a Resource Gateway in your VPC, creating one ENI per subnet you specified.</p> 
<p>The following architecture diagram shows the network flow.</p> 
<p><img alt="" class="alignnone size-full wp-image-129873" height="516" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-3-8.png" width="1650" /></p> 
<p>AgentCore Gateway initiates the request and routes it to the Resource Gateway provisioned inside Resource Owner VPC. Traffic passes through the ENI inside your private subnet, where your security group rules govern what reaches the next hop. From there, the request reaches the execute-api VPC endpoint, which provides private connectivity to your Amazon API Gateway internal endpoint. The endpoint URL format https://{api-id}-{vpce-id}.execute-api.{region}.amazonaws.com/{stage} is what you provided in the openApiSchema field of your CreateGatewayTarget call.</p> 
<h3>Private MCP server on Amazon EKS</h3> 
<p>In this section, you create an AgentCore Gateway target that routes to a private MCP server running on Amazon EKS. Call the CreateGatewayTarget API with the following parameters. In the mcpServer block, provide your internal MCP server endpoint URL. In the managedVpcResource block, provide your VPC ID, subnet IDs, and security group ID.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-clike">  aws bedrock-agentcore-control create-gateway-target \
    --region us-west-2 \
    --cli-input-json '{
      "gatewayIdentifier": "&lt;GATEWAY_ID&gt;",           
      "name": "private-apigw",
      "description": "Private API Gateway",                 
      "targetConfiguration": {                     
        "mcp": {
          "mcpServer": {                     
            "endpoint": "https://internal.example.com/csm/mcp"
          }
        }
      },
      "credentialProviderConfigurations": [...],          
      "privateEndpoint": {
        "managedVpcResource": {
          "vpcIdentifier": "&lt;VPC_ID&gt;",
          "subnetIds": ["&lt;SUBNET_ID_1&gt;", "&lt;SUBNET_ID_2&gt;"],
          "endpointIpAddressType": "IPV4",
          "securityGroupIds": ["&lt;VPCE_SG_ID&gt;"]
        }                                        
      }           
    }'
</code></pre> 
</div> 
<p>After you run this command, AgentCore provisions a Resource Gateway in your VPC, creating one ENI per subnet you specified. The following architecture diagram shows the end-to-end traffic path.</p> 
<p><img alt="" class="alignnone size-full wp-image-129874" height="707" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-4-8.png" width="1650" /></p> 
<p>AgentCore Gateway sends an HTTPS request to your internal endpoint. The Amazon Route 53 private hosted zone resolves that domain to the internal <a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html#:~:text=Classic%20Load%20Balancers.-,Network%20Load%20Balancer%20components,listeners%20to%20your%20load%20balancer." rel="noopener noreferrer" target="_blank">Network Load Balancer</a> (NLB). The request enters the Resource Owner VPC through the Resource Gateway, passes through the ENI governed by your security groups, and arrives at the NLB. The NLB terminates TLS on port 443 using an <a href="https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html" rel="noopener noreferrer" target="_blank">AWS Certificate Manager (ACM) public certificate</a>, then forwards the request over HTTP on port 80 to the<a href="https://docs.nginx.com/nginx-ingress-controller/" rel="noopener noreferrer" target="_blank"> NGINX Ingress Controller</a> running on Amazon EKS, which routes it to the appropriate pod.</p> 
<h3>Private REST API target</h3> 
<p>In this section, you create an AgentCore Gateway target that routes to a private REST API endpoint. This applies to any REST API running inside your Amazon VPC such as a containerized microservice. The CreateGatewayTarget API call follows the same pattern as the previous sections. In the openApiSchema field, provide your OpenAPI schema describing the REST API. In the <code>managedVpcResource</code> block, provide your VPC ID, subnet IDs, and security group ID. After AgentCore Gateway provisions the Resource Gateway in your VPC, the following architecture diagram shows the end-to-end traffic path.</p> 
<p><img alt="" class="alignnone size-full wp-image-129875" height="792" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-5-8.png" width="1648" /></p> 
<p>AgentCore Gateway sends an HTTPS request to your internal endpoint. The Amazon Route 53 private hosted zone resolves that domain to the internal <a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html" rel="noopener noreferrer" target="_blank">Application Load Balancer</a> (ALB). The request enters the Resource Owner VPC through the Resource Gateway, passes through the ENI governed by your security groups, and arrives at the internal ALB. The ALB terminates TLS on port 443 using an AWS Certificate Manager (ACM) public certificate, then forwards the request over HTTP on port 8000 to the target group containing your backend servers.</p> 
<h2>Clean up</h2> 
<p>To avoid ongoing charges, delete all resources created in this walkthrough. For reference, see the AgentCore Gateway VPC egress <a href="https://aws.amazon.com/bedrock/agentcore/pricing/" rel="noopener noreferrer" target="_blank">pricing page</a>. Additionally, Amazon EKS clusters, load balancers, and API Gateway endpoints incur charges while running. Verify that your resources are deleted to stop charges. If you followed the GitHub <a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/02-AgentCore-gateway/16-vpc-egress" rel="noopener noreferrer" target="_blank">sample</a>, make sure to run the cleanup section at the end of each Jupyter Notebook.</p> 
<p>If you used managed VPC resource mode, deleting the Gateway Target removes the associated Amazon VPC Resource Gateway.</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws bedrock-agentcore delete-gateway-target \
    --gateway-identifier &lt;gateway-id&gt; \
    --target-id &lt;target-id&gt;</code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>As AI agents take on more complex tasks, they need secure access to the tools and services that power your business, many of which live inside private networks. AgentCore Gateway VPC egress allows your agents to reach private MCP servers, internal APIs, databases, and on-premises systems without exposing them to the public internet.Managed VPC resource mode integrates directly with your existing VPC and requires minimal configuration. Self-managed lattice resource mode gives you fine-grained control but requires additional setup. Both route traffic through a Resource Gateway that doesn’t leave the AWS network.</p> 
<h2>Next steps</h2> 
<ul> 
 <li>Identify one internal API or MCP server in your environment that would benefit from AI agent access</li> 
 <li>Review your existing Amazon VPC architecture to determine which mode (Managed VPC Resource or Self-Managed Lattice Resource) fits your requirements</li> 
 <li>Review the Amazon Bedrock AgentCore Gateway <a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-vpc-egress.html" rel="noopener noreferrer" target="_blank">documentation </a>for additional configuration options</li> 
 <li>Explore the Amazon VPC Lattice Resource Gateway <a href="https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/vpc-egress-private-endpoints.html" rel="noopener noreferrer" target="_blank">documentation </a>for cross-account scenarios</li> 
 <li>Explore additional integration patterns and advanced configurations, see <a href="https://github.com/awslabs/agentcore-samples/tree/main/01-tutorials/02-AgentCore-gateway/16-vpc-egress" rel="noopener noreferrer" target="_blank">GitHub samples.</a></li> 
</ul> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><strong><img alt="" class="alignleft size-full wp-image-129895" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-6-5.png" width="100" /></strong><a href="https://www.linkedin.com/in/eashan-kaushik/" rel="noopener" target="_blank">Eashan Kaushik</a> is a Specialist Solutions Architect AI/ML at Amazon Web Services. He is driven by creating cutting-edge generative AI solutions while prioritizing a customer-centric approach to his work. Before this role, he obtained an MS in Computer Science from NYU Tandon School of Engineering. Outside of work, he enjoys sports, lifting, and running marathons.</p> 
<p style="clear: both;"><a href="https://www.linkedin.com/in/thomasmathewv/" rel="noopener noreferrer" target="_blank"><img alt="" class="alignleft size-full wp-image-129897" height="132" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-7-5.png" width="100" />Thomas Mathew Veppumthara</a> is a Sr. Software Engineer at Amazon Web Services (AWS) with Amazon Bedrock AgentCore. He has previous generative AI leadership experience in Amazon Bedrock Agents and nearly a decade of distributed systems expertise across Amazon eCommerce Services and Amazon Elastic Block Store (Amazon EBS). He holds multiple patents in distributed systems, storage, and generative AI technologies.</p> 
<p style="clear: both;"><a href="https://www.linkedin.com/in/rohin-meduri-bb3b2113b/" rel="noopener noreferrer" target="_blank">Rohin Meduri</a><a href="https://www.linkedin.com/in/rohin-meduri-bb3b2113b/" rel="noopener noreferrer" target="_blank"><img alt="" class="alignleft wp-image-129899 size-thumbnail" height="137" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/04/29/image-8-5-100x137.png" width="100" /></a> is a Software Engineer at Amazon Web Services (AWS) with Amazon Bedrock AgentCore. He has previous AI development experience with Amazon Bedrock Agents and Amazon Lex. Before this role, he obtained a BS in Computer Science from the University of Washington. Outside of work, he enjoys chess, pool, and music production.</p>
