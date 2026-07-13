---
title: "Securing Amazon Bedrock AgentCore Runtime with AWS WAF"
url: "https://aws.amazon.com/blogs/machine-learning/securing-amazon-bedrock-agentcore-runtime-with-aws-waf/"
date: "2026-07-08"
author: "Puneeth Komaragiri"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
This post shows you two architecture patterns that address this problem. Both use an internet-facing ALB with AWS WAF and route traffic through a VPC Interface Endpoint to AgentCore Runtime. Pattern 1 places an AWS Lambda proxy between the ALB and the VPC Endpoint, giving you full control over request transformation.
