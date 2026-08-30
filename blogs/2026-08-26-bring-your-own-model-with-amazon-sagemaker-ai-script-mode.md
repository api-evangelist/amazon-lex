---
title: "Bring your own model with Amazon SageMaker AI: Script mode in SDK v3"
url: "https://aws.amazon.com/blogs/machine-learning/bring-your-own-model-with-amazon-sagemaker-ai-script-mode-in-sdk-v3/"
date: "2026-08-26"
author: "Bobby Lindsey"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
The SageMaker Python SDK v3 redesigns script mode with unified ModelTrainer and ModelBuilder classes. This post walks through two end-to-end examples, a scikit-learn Random Forest and a multi-GPU Stable Diffusion 3.5 LoRA fine-tune, showing how SourceCode syncs your local code into any container at runtime so you can iterate without rebuilding Docker images.
