---
title: "Week 8: Deploying and completing FoodieRecipe"
date: 2026-08-07T00:00:00+07:00
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

## 1. Objectives

- Deploy Next.js and NestJS with Docker on EC2 behind Nginx and a custom domain.
- Complete the Next.js–NestJS–S3–Rekognition–Bedrock–CloudFront pipeline.
- Configure DNS and HTTPS, then verify production access.
- Collect application and RDS logs with Amazon CloudWatch.
- Test sign-in, search, likes, comments, and AI-assisted recipe creation end to end.
- Review security, restart behavior, rollback, and handover documentation.

## 2. Work plan

> **Week 8 schedule:** Monday, 10/08/2026 – Friday, 14/08/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Build the Next.js container, run it with NestJS on EC2, and configure Nginx | Run the Frontend and Backend in production |
| Tuesday | Configure DNS and HTTPS, then verify CloudFront image delivery | Serve the custom domain and CDN images reliably |
| Wednesday | Configure CloudWatch Logs and inspect application/RDS logs | Enable monitoring and troubleshooting |
| Thursday | Test sign-in, image upload, Rekognition, and Bedrock | Verify the complete AI pipeline |
| Friday | Test likes, comments, restart, rollback, and finish documentation | Prepare the product for handover |

## 3. Work completed

### 3.1. Deploying the website with a custom domain

The Frontend under `web` uses the Next.js standalone output and is packaged with Docker. The production API URL and CloudFront image domain are supplied as build arguments. The container exposes port `3000` only on loopback and Nginx serves it externally over HTTPS.

The Dockerfile must accept the CloudFront domain before `pnpm build` because `NEXT_PUBLIC_*` values are embedded into the client at build time:

```dockerfile
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}

ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN
ENV NEXT_PUBLIC_CLOUDFRONT_DOMAIN=${NEXT_PUBLIC_CLOUDFRONT_DOMAIN}

RUN pnpm build
```

```bash
cd web

docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://myapps.io.vn/api \
  --build-arg NEXT_PUBLIC_CLOUDFRONT_DOMAIN=<CLOUDFRONT_DOMAIN> \
  -t foodierecipe-web:production .

docker rm -f foodierecipe-web 2>/dev/null || true
docker run -d \
  --name foodierecipe-web \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  foodierecipe-web:production

curl -I http://127.0.0.1:3000
```

Nginx proxies interface requests to Next.js and `/api/` requests to NestJS. The production result is verified through the HTTPS custom domain:

![FoodieRecipe running on myapps.io.vn](/images/1-Worklog/1.8-Week8/production-custom-domain.png)

### 3.2. Configuring DNS and HTTPS as code

Create an `A` record for `myapps.io.vn` pointing to the EC2 Elastic IP. The following example uses Amazon Route 53:

```json
{
  "Comment": "Point FoodieRecipe domain to the EC2 Elastic IP",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "myapps.io.vn.",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{ "Value": "<EC2_ELASTIC_IP>" }]
      }
    }
  ]
}
```

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id <HOSTED_ZONE_ID> \
  --change-batch file://dns-change.json

dig +short myapps.io.vn
sudo certbot --nginx -d myapps.io.vn
curl -I https://myapps.io.vn
```

If another provider manages DNS, create the equivalent `A` record and keep the Nginx/HTTPS configuration on EC2.

### 3.3. Verifying CloudFront as code

CloudFront delivers images from the private S3 bucket; it does not replace the Next.js container. The distribution uses an S3 REST origin, Origin Access Control, HTTPS, and a cache policy. Verify it without a Console screenshot:

```bash
aws cloudfront get-distribution \
  --id <DISTRIBUTION_ID> \
  --query '{Status:Distribution.Status,Domain:Distribution.DomainName,Aliases:Distribution.DistributionConfig.Aliases.Items,Origins:Distribution.DistributionConfig.Origins.Items[*].{Domain:DomainName,OAC:OriginAccessControlId},ViewerProtocol:Distribution.DistributionConfig.DefaultCacheBehavior.ViewerProtocolPolicy}'

aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths '/delivery/*'

curl -I '<SIGNED_CLOUDFRONT_IMAGE_URL>'
```

The expected result is a `Deployed` distribution, a signed URL returning `200`, `X-Cache: Hit from cloudfront` on repeated requests, and `403 AccessDenied` for direct S3 access.

### 3.4. Testing the AI pipeline end to end

**Step 1 – The user signs in to FoodieRecipe:**

![FoodieRecipe sign-in interface](/images/1-Worklog/1.4-Week4/login-page.png)

**Step 2 – The user selects an ingredient image:**

![Selected ingredient image](/images/1-Worklog/1.5-Week5/ai-image-selected.png)

**Step 3 – NestJS uploads the image to S3 and calls Rekognition DetectLabels:**

![Labels detected by Rekognition](/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

The Backend filters labels by confidence, normalizes them, and removes duplicates before sending the ingredient list to Bedrock.

**Step 4 – Bedrock processes the ingredient list:**

![Bedrock generating a recipe](/images/1-Worklog/1.6-Week6/bedrock-processing.png)

**Step 5 – The application displays the generated recipe:**

![Recipe generated by Bedrock](/images/1-Worklog/1.6-Week6/bedrock-recipe-result.png)

**Step 6 – The user views ingredients and preparation steps:**

![Generated recipe details](/images/1-Worklog/1.6-Week6/generated-recipe-details.png)

After the AI workflow, recipe search, like/unlike, and comment creation/deletion are tested to verify that Next.js, NestJS, and RDS remain synchronized in production.

### 3.5. CloudWatch Logs and monitoring

CloudWatch contains the `foodie-recipe-log` log group for application logs and `RDSOSMetrics` for RDS Enhanced Monitoring. Retention periods are selected to support troubleshooting without keeping logs unnecessarily long.

![FoodieRecipe CloudWatch Log Groups](/images/1-Worklog/1.8-Week8/cloudwatch-log-groups.png)

Logs are inspected after sign-in, AI recipe generation, and failed API calls. They must not expose AWS credentials, JWTs, complete signed URLs, or Secrets Manager values.

### 3.6. Operational checks and handover

- Reboot EC2 and confirm that Docker and Nginx restart automatically.
- Verify the health endpoint, RDS connectivity, and Rekognition/Bedrock access after restart.
- Roll back to the previous stable image tag when a new release fails its health check.
- Review the IAM role for least privilege, S3 Block Public Access, OAC, and Security Groups.
- Test `https://myapps.io.vn`, the API, CloudFront images, likes, and comments.
- Complete the architecture, API contract, deployment, rollback, and troubleshooting documents.

## 4. Knowledge and skills gained

- Deployed standalone Next.js and NestJS containers on EC2.
- Configured Nginx, DNS, and HTTPS for a custom domain.
- Verified a private S3 origin and CloudFront caching.
- Monitored application and RDS logs with Amazon CloudWatch.
- Tested the complete S3–Rekognition–Bedrock–CloudFront pipeline.
- Reviewed security, restart, rollback, and product operations documentation.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| The domain did not resolve to the server | Checked the `A` record, nameservers, and DNS propagation |
| The site used HTTPS but API or images were blocked | Used HTTPS consistently and checked CORS and production build URLs |
| CloudFront returned `403` | Checked the signed URL, OAC, bucket policy, and object key |
| New logs did not appear in CloudWatch | Checked CloudWatch Agent, the IAM role, log paths, and Region |
| A new container failed its health check | Inspected logs and migrations, then rolled back to a stable image tag |
| The AI response did not match the schema | Validated Bedrock output and recorded failure for bounded retries |

## 6. Final results

- FoodieRecipe runs at `https://myapps.io.vn`, with Next.js and NestJS Docker containers on EC2 behind Nginx.
- Sign-in, recipe management, search, likes, and comments work in production.
- Images are stored in a private S3 bucket, recognized with Rekognition, and delivered through CloudFront.
- Bedrock generates recipe content from normalized ingredient labels.
- CloudWatch collects application and RDS logs for monitoring and troubleshooting.
- DNS, HTTPS, deployment, restart, rollback, and handover procedures are complete.

## 7. References

- [Self-hosting Next.js](https://nextjs.org/docs/app/guides/self-hosting)
- [Creating and managing Amazon Route 53 records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-creating.html)
- [Using Certbot](https://eff-certbot.readthedocs.io/en/stable/using.html)
- [Collecting metrics, logs, and traces with the CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Configuring restart policies for Docker containers](https://docs.docker.com/engine/containers/start-containers-automatically/)
- [Operational excellence in the AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/operational-excellence.html)
