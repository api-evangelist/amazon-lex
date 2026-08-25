---
title: "Reduce RAG costs on Amazon Bedrock with query-aware compression"
url: "https://aws.amazon.com/blogs/machine-learning/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compression/"
date: "2026-08-21"
author: "Aakanksha Veesam"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
Input tokens are often a meaningful part of the cost of running Retrieval Augmented Generation (RAG) at scale. This post describes a query-aware context compression pattern on Amazon Bedrock: after retrieval, a smaller model filters retrieved chunks against the query before the primary model answers, reducing input tokens and cost while preserving answer quality.
