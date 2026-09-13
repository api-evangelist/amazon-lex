---
title: "Reduce LLM latency with prefix-aware routing on Amazon SageMaker Inference"
url: "https://aws.amazon.com/blogs/machine-learning/reduce-llm-latency-with-prefix-aware-routing-on-amazon-sagemaker-inference/"
date: "2026-09-10"
author: "Kareem Syed-Mohammed"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
Amazon SageMaker Inference now offers prefix-aware routing, a routing strategy that sends requests sharing the same prompt prefix to the same instance so the KV cache stays warm. In benchmarks on Llama 3.1 70B, it reduced P50 time-to-first-token by up to 77% and raised KV cache hit rates from about 25% to over 80%.
