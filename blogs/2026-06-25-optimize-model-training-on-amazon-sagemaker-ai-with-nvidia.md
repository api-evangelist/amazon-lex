---
title: "Optimize model training on Amazon SageMaker AI with NVIDIA Blackwell"
url: "https://aws.amazon.com/blogs/machine-learning/optimize-model-training-on-amazon-sagemaker-ai-with-nvidia-blackwell/"
date: "2026-06-25"
author: "Andrea Gallo"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
This post shows you how to configure training jobs on Amazon SageMaker AI to get the most out of Blackwell’s architecture on AWS. You learn how to select batch sizes and sequence lengths that take advantage of Blackwell’s expanded memory, choose the right precision format for your model size (1B to 64B parameters), and apply activation checkpointing strategically. By the end, you have a practical framework for tuning your training configuration and launching distributed training jobs on P6-B200 instances.
