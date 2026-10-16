---
title: "Automate remediation post AWS DevOps Agent investigation"
url: "https://aws.amazon.com/blogs/machine-learning/automate-remediation-post-aws-devops-agent-investigation/"
date: "2026-10-07"
author: "Michele Scarimbolo"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
AWS DevOps Agent can diagnose production incidents but is kept in observe-and-report mode so it does not change resources directly. This post shows how to use AWS Lambda Durable Functions, Amazon EventBridge, and Amazon Bedrock to turn its investigation summaries into pre-validated fixes an on-call engineer can approve with a single action.
