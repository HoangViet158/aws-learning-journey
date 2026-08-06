---
title: "Week 7: Faster Image Access with Amazon CloudFront"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

## 1. Objectives

- Deliver FoodieRecipe images from S3 through CloudFront.
- Keep the bucket private and control CloudFront read access.
- Design a cache policy for versioned images.
- Measure and improve image-loading speed in Next.js.

## 2. Work plan

> **Week 7 schedule:** Monday, 03/08/2026 – Friday, 07/08/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study edge locations, cache keys, TTL, and invalidation | Understand CDN behavior |
| Tuesday | Create a distribution with an S3 origin and Origin Access Control | Keep the bucket private |
| Wednesday | Configure cache policy, compression, and response headers | Reduce requests to S3 |
| Thursday | Integrate CloudFront URLs into NestJS and Next.js | Load images through the CDN |
| Friday | Measure cache hits and latency and test image replacement/deletion | Confirm the performance improvement |

## 3. Work completed

### 3.1. Private S3 origin

- Kept Block Public Access enabled on the bucket.
- Used Origin Access Control and a bucket policy allowing CloudFront reads.
- Avoided returning direct S3 URLs to users.
- Limited the image behavior to `GET` and `HEAD` methods.

### 3.2. Cache strategy

- Used versioned or hashed object keys when images changed.
- Added suitable `Cache-Control` headers for stable content.
- Removed unnecessary query strings from the cache key.
- Reserved invalidations for cases where the object key could not change.

### 3.3. Next.js integration

NestJS returns a CloudFront URL from the object key after the image is ready. Next.js uses the URL with suitable dimensions and lazy loading to reduce initial page data.

## 4. Knowledge and skills gained

- Understood CloudFront distributions, origins, behaviors, and edge caching.
- Protected an S3 origin with Origin Access Control.
- Designed cache keys, TTLs, and versioned object keys.
- Measured cache hit/miss behavior and image-loading time.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| CloudFront returned 403 for an existing object | Checked OAC, bucket policy, and object key |
| An old image remained cached | Used a version/hash in the key or a targeted invalidation |
| Cache hit ratio was low | Reduced query and header variation in the cache key |

## 6. Deliverables

- A CloudFront distribution reading from a private S3 bucket.
- CloudFront image URLs integrated into NestJS and Next.js.
- A cache policy and object-versioning strategy improving access speed.

## 7. Next-week plan

Integrate the complete pipeline, test it, optimize costs, and finish documentation.
