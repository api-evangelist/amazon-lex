---
title: "Tiered KV cache for large LLMs on Amazon SageMaker HyperPod with Curvine"
url: "https://aws.amazon.com/blogs/machine-learning/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine/"
date: "2026-08-12"
author: "Qingyuan Tang"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
Running large language model inference at scale forces a KV cache trade-off: oversized GPU instances or slow time-to-first-token. This post builds a tiered KV cache on Amazon SageMaker HyperPod that extends the cache into a shared, distributed NVMe pool with Curvine, so replicas reuse cache at near-local-disk speeds on cost-efficient instances.
