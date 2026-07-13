---
title: "Build a semantic layer for agentic AI on AWS with Stardog and Amazon Bedrock AgentCore"
url: "https://aws.amazon.com/blogs/machine-learning/build-a-semantic-layer-for-agentic-ai-on-aws-with-stardog-and-amazon-bedrock-agentcore/"
date: "2026-07-10"
author: "Navin Sharma"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
In this post we show how to build a semantic layer on AWS using Stardog’s Semantic AI Application over Amazon Aurora and Amazon Redshift, and how to run a Strands Agents agent on Amazon Bedrock AgentCore that queries the layer to answer customer 360 questions across both sources without extract, transform, and load (ETL). The same Stardog deployment works behind AWS computes (Amazon Elastic Kubernetes Service (Amazon EKS), Amazon Elastic Container Service (Amazon ECS), and AWS Lambda). We use AgentCore here because it bundles inbound auth, hosting, and tool credentials into one managed service
