---
title: "Capacity-aware inference: Automatic instance fallback for SageMaker AI endpoints"
url: "https://aws.amazon.com/blogs/machine-learning/capacity-aware-inference-automatic-instance-fallback-for-sagemaker-ai-endpoints/"
date: "2026-05-04"
author: "Kareem Syed-Mohammed"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
As organizations scale generative AI workloads in production, securing reliable GPU compute has become one of the most persistent operational challenges. Large language models (LLMs) and multimodal architectures demand specific instance types and when that capacity isn’t available, endpoints fail before they serve a single request. Building a real-time inference endpoint on Amazon SageMaker AI has meant committing to a single instance type at creation time. When that type had insufficient capacity, the endpoint failed to reach a running state.
