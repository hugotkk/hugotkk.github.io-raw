---
title: "Critical Review of Hit Discussion's Video Monitoring Service: Pitfalls of Blaming Microservices"
date: 2023-05-07
tags:
- aws
- case-study
---

A recent article on Prime Video Tech describes their approach to scaling up their audio-video monitoring service and reducing costs by 90%.

This blog post aims to critically analyze the article and address some issues and misconceptions.

The Service and the Problem

- The monitoring service converts video streams into video/audio formats and stores them on Amazon S3.
- Multiple detectors identify problems in the video, and the results are saved back to S3.
- Escalating costs were due to data transfer between S3, orchestration with AWS Step Functions, and not being able to use EC2 Savings Plans.
- Detectors had to download the same data multiple times, causing high costs and inefficient resource use.
- Scaling bottleneck: orchestration management using AWS Step Functions led to reaching account limits and charges per state transition.

My View:

- The post's motivation is valid as it shares a story about addressing cost and scaling issues in a Prime Video service.
- However, bloggers and YouTubers used the article to create headlines pitting microservices against monolithic architectures.
- The real issue is not about microservices vs. monoliths, but the pricing model on AWS and understanding the context of the solution.

- The post is missing information: size of video streams, detector runtime, and service profile (CPU, memory, concurrent users, usage patterns, etc.).
- These factors are essential to understanding scalability and cost implications.
- The solution involves vertical scaling, which can be more expensive and challenging to adjust based on usage patterns, possibly leading to underutilization during off-peak times.

My Advice/Suggestion:

- Consider trade-offs, limitations, and applicability of the solution.
- Understand that AWS is not synonymous with microservices; the real issue is its pricing model.
- Know your service's requirements and usage patterns before making architectural decisions.
- Take a balanced approach and make informed decisions based on the specific needs of your application, rather than being swayed by sensational headlines.
