---
title: "Multi-tenant LLM analytics with row-level security: How we built a secure agent on AWS"
url: "https://aws.amazon.com/blogs/machine-learning/multi-tenant-llm-analytics-with-row-level-security-how-we-built-a-secure-agent-on-aws/"
date: "2026-06-29"
author: "Anuranjan Mondal"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
In this post, we show you how PAR built a production-ready multi-tenant LLM analytics system that enforces row-level security through a three-layer architecture: cryptographic request signing with AWS SigV4, semantic validation on Amazon Bedrock, and programmatic data isolation via Split-Plane SQL. We demonstrate how each layer operates independently to reduce the risk of cross-tenant data exposur
