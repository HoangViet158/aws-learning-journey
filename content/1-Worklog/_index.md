---
title: "Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Overview

This worklog documents an eight-week journey of building the image-processing feature for **FoodieRecipe**—a web application where users can discover, publish, and manage cooking recipes. The Frontend uses **Next.js** and the Backend uses **NestJS**. Images are stored in **Amazon S3**, analyzed with **Amazon Rekognition** and **Amazon Bedrock**, and delivered through **Amazon CloudFront** for faster access.

| Week | Topic | Details |
| :--: | ----- | ------- |
| **Week 1** | AWS foundations and FoodieRecipe design | [Explore AWS, IAM, and Budget Alerts; analyze requirements and design the image pipeline](1.1-week1/) |
| **Week 2** | Amazon S3 image management | [Design buckets, object keys, permissions, metadata, and the image-upload flow](1.2-week2/) |
| **Week 3** | NestJS Backend and S3 | [Build upload APIs, validation, pre-signed URLs, and image-status management](1.3-week3/) |
| **Week 4** | Next.js Frontend | [Build upload, preview, progress, and NestJS integration experiences](1.4-week4/) |
| **Week 5** | Amazon Rekognition | [Detect labels, moderate content, and extract information from food images](1.5-week5/) |
| **Week 6** | Amazon Bedrock | [Use a multimodal AI model to analyze images and suggest recipe content](1.6-week6/) |
| **Week 7** | Amazon CloudFront | [Deliver S3 images through a CDN, configure caching, and improve access speed](1.7-week7/) |
| **Week 8** | Integration and completion | [Complete the S3–Rekognition–Bedrock–CloudFront pipeline, test it, and document the work](1.8-week8/) |
