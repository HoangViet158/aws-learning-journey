---
title: "Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Overview

This worklog documents eight weeks of independently building and deploying the complete **FoodieRecipe** application, where users can register, sign in, discover, publish, manage, like, and comment on cooking recipes. The Frontend uses **Next.js**, the Backend uses **NestJS**, and business data is stored in **Amazon RDS for PostgreSQL**. Images are stored in **Amazon S3**, analyzed with **Amazon Rekognition** and **Amazon Bedrock**, and delivered through **Amazon CloudFront**. The Dockerized Backend runs on EC2 behind Nginx, while IAM, Secrets Manager, and CloudWatch provide security, secret management, and observability.

| Week | Topic | Details |
| :--: | ----- | ------- |
| **Week 1** | AWS foundations and FoodieRecipe design | [Explore AWS, IAM, and Budget Alerts; analyze requirements and design the image pipeline](1.1-week1/) |
| **Week 2** | S3, RDS, and data security | [Design image/web buckets, the PostgreSQL model, IAM, and Secrets Manager](1.2-week2/) |
| **Week 3** | NestJS Backend | [Build authentication, recipe, like, comment, image-upload APIs, and the Dockerfile](1.3-week3/) |
| **Week 4** | Next.js Frontend | [Build account, recipe, interaction, search, image-upload, and NestJS integration experiences](1.4-week4/) |
| **Week 5** | Amazon Rekognition | [Detect labels, moderate content, and extract information from food images](1.5-week5/) |
| **Week 6** | Amazon Bedrock | [Use a multimodal AI model to analyze images and suggest recipe content](1.6-week6/) |
| **Week 7** | AWS deployment and CloudFront | [Create EC2/RDS, deploy NestJS with Docker/Nginx, and configure CloudFront](1.7-week7/) |
| **Week 8** | Product deployment and completion | [Deploy Next.js, configure CloudWatch, perform end-to-end tests, and complete documentation](1.8-week8/) |
