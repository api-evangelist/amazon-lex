---
title: "Fault tolerant distributed training on Amazon EKS using NVRx"
url: "https://aws.amazon.com/blogs/machine-learning/fault-tolerant-distributed-training-on-amazon-eks-using-nvrx/"
date: "2026-09-16"
author: "Aravind Neelakantan"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
Integrate NVIDIA Resiliency Extension (NVRx) into PyTorch FSDP training on Amazon EKS to overlap checkpoint I/O with training and recover from GPU faults in seconds. This post covers async checkpointing, in-process restart, and ft_launcher in-job restart, with H100 benchmarks at 2 to 8 nodes showing 99%+ training efficiency and second-scale recovery.
