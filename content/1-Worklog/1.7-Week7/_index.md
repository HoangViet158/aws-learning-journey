---
title: "Week 7: AWS Deployment and Amazon CloudFront"
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
- Create private Amazon RDS, EC2, Security Groups, an IAM role, and Secrets Manager.
- Deploy NestJS with Docker, run migrations, and configure the Nginx reverse proxy.

## 2. Work plan

> **Week 7 schedule:** Monday, 03/08/2026 – Friday, 07/08/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Create private RDS, a DB subnet group, Security Groups, and a secret | Prepare the database for the Backend |
| Tuesday | Create EC2, attach the IAM role, and install Docker/Nginx/CloudWatch Agent | Prepare the server for deployment |
| Wednesday | Build the container, run migrations, and configure Nginx | Run NestJS on EC2 |
| Thursday | Create CloudFront with an S3 origin and Origin Access Control | Deliver private images through the CDN |
| Friday | Integrate URLs, measure cache hit/latency, and test restart behavior | Verify deployment and performance |

## 3. Work completed

### 3.1. Overall deployment architecture

The FoodieRecipe architecture combines Next.js, NestJS, and AWS services into a complete application. CloudFront delivers content and images from S3, while Next.js calls the NestJS API running in Docker behind Nginx on EC2. The Backend connects to PostgreSQL on RDS, invokes Rekognition to identify ingredients, and uses Bedrock to generate recipes. Secrets Manager protects application secrets, IAM controls access, and CloudWatch collects system metrics and logs.

![FoodieRecipe deployment architecture on AWS](/images/1-Worklog/1.7-Week7/foodierecipe-aws-architecture.png)

The main flows in the diagram are:

1. Users access content through Amazon CloudFront.
2. CloudFront retrieves and caches content from Amazon S3.
3. Next.js sends application requests to NestJS on EC2.
4. Images are uploaded to the private S3 bucket through a controlled upload flow.
5. The Backend processes object information and stores image metadata.
6. NestJS reads and writes FoodieRecipe data in Amazon RDS for PostgreSQL.
7. Amazon Bedrock generates recipe content from the detected ingredients.
8. Amazon Rekognition detects labels and ingredients in uploaded images.
9. EC2 retrieves application secrets from AWS Secrets Manager through its IAM role.

### 3.2. Creating EC2 and attaching an IAM role

Create a Linux EC2 instance in a public subnet and attach an Elastic IP, the `DeployEC2` IAM role, and the required Security Groups. Allow SSH `22` only from the administrator IP, expose HTTP `80`/HTTPS `443`, and keep NestJS port `3001` private. RDS accepts PostgreSQL `5432` only from the Backend Security Group.

**The FoodieRecipe EC2 instance is running and passes all status checks:**

![FoodieRecipe EC2 instance in the Running state](/images/1-Worklog/1.7-Week7/ec2-instance-running.png)

**An IAM role is attached so EC2 receives temporary credentials:**

![FoodieRecipe EC2 IAM role and Security Groups](/images/1-Worklog/1.7-Week7/ec2-iam-role.png)

After connecting through SSH, verify the IAM identity and install deployment tools:

```bash
aws sts get-caller-identity

sudo dnf update -y
sudo dnf install -y git docker nginx jq
sudo systemctl enable --now docker
sudo systemctl enable --now nginx
sudo usermod -aG docker ec2-user
```

Sign out and reconnect for Docker group membership to take effect. The IAM role grants only the required S3, Rekognition, Bedrock, Secrets Manager, and CloudWatch actions; no access key is stored on EC2.

### 3.3. Building, migrating, and running NestJS with Docker

The `api` Dockerfile uses Node.js 22, installs dependencies with pnpm, generates Prisma Client, builds NestJS, and runs `prisma migrate deploy` before starting `node dist/main.js`.

```bash
git clone https://github.com/<owner>/<repository>.git foodierecipe
cd foodierecipe/api
docker build -t foodierecipe-api:week7 .
```

No `.env.production` file is created. Non-sensitive configuration is stored in Systems Manager Parameter Store, while sensitive values are stored as JSON in Secrets Manager:

```text
Parameter Store
├── /foodierecipe/prod/PORT
├── /foodierecipe/prod/AWS_BUCKET_NAME
├── /foodierecipe/prod/BEDROCK_MODEL_ID
├── /foodierecipe/prod/CLOUDFRONT_DOMAIN
├── /foodierecipe/prod/CLOUDFRONT_KEY_PAIR_ID
└── /foodierecipe/prod/CLOUDFRONT_URL_EXPIRES_IN

Secrets Manager
├── prod/foodie-recipe/db
│   └── DATABASE_URL
└── prod/foodie-recipe/app
    ├── JWT_SECRET
    └── CLOUDFRONT_PRIVATE_KEY_BASE64
```

The `DeployEC2` IAM role needs `ssm:GetParameter`, `secretsmanager:GetSecretValue`, and, when a secret uses a customer-managed KMS key, `kms:Decrypt` for only the required ARNs. The deployment script retrieves values with temporary IAM role credentials, injects them into the container, and clears its temporary shell variables without creating a secret file on disk:

```bash
#!/usr/bin/env bash
set -euo pipefail

REGION="ap-southeast-1"
PARAMETER_PATH="/foodierecipe/prod"

get_parameter() {
  aws ssm get-parameter \
    --region "$REGION" \
    --name "$PARAMETER_PATH/$1" \
    --query 'Parameter.Value' \
    --output text
}

DB_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id "prod/foodie-recipe/db" \
  --query SecretString \
  --output text)

APP_SECRET=$(aws secretsmanager get-secret-value \
  --region "$REGION" \
  --secret-id "prod/foodie-recipe/app" \
  --query SecretString \
  --output text)

DATABASE_URL=$(jq -r '.DATABASE_URL' <<< "$DB_SECRET")
JWT_SECRET=$(jq -r '.JWT_SECRET' <<< "$APP_SECRET")
CLOUDFRONT_PRIVATE_KEY_BASE64=$(jq -r '.CLOUDFRONT_PRIVATE_KEY_BASE64' <<< "$APP_SECRET")

docker rm -f foodierecipe-api 2>/dev/null || true
docker run -d \
  --name foodierecipe-api \
  --restart unless-stopped \
  -p 127.0.0.1:3001:3001 \
  -e "DATABASE_URL=$DATABASE_URL" \
  -e "PORT=$(get_parameter PORT)" \
  -e "NODE_ENV=production" \
  -e "COOKIE_SECURE=true" \
  -e "JWT_SECRET=$JWT_SECRET" \
  -e "JWT_EXPIRES_IN=7d" \
  -e "AWS_REGION=$REGION" \
  -e "AWS_BUCKET_NAME=$(get_parameter AWS_BUCKET_NAME)" \
  -e "BEDROCK_MODEL_ID=$(get_parameter BEDROCK_MODEL_ID)" \
  -e "CLOUDFRONT_DOMAIN=$(get_parameter CLOUDFRONT_DOMAIN)" \
  -e "CLOUDFRONT_KEY_PAIR_ID=$(get_parameter CLOUDFRONT_KEY_PAIR_ID)" \
  -e "CLOUDFRONT_PRIVATE_KEY_BASE64=$CLOUDFRONT_PRIVATE_KEY_BASE64" \
  -e "CLOUDFRONT_URL_EXPIRES_IN=$(get_parameter CLOUDFRONT_URL_EXPIRES_IN)" \
  foodierecipe-api:week7

unset DB_SECRET APP_SECRET DATABASE_URL JWT_SECRET CLOUDFRONT_PRIVATE_KEY_BASE64

docker ps
docker logs --tail 100 foodierecipe-api
curl http://127.0.0.1:3001/api
```

The container logs must show successful migrations and NestJS startup. Verify the RDS connection through registration, recipe creation, or a Prisma query instead of exposing the database port publicly.

### 3.4. Configuring the Nginx reverse proxy

Create `/etc/nginx/conf.d/foodierecipe.conf`:

```nginx
server {
    listen 80;
    server_name _;

    client_max_body_size 10m;

    location /api/ {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Validate the configuration and call the API through Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
curl http://127.0.0.1/api
curl http://<EC2_PUBLIC_IP>/api
```

### 3.5. Creating CloudFront OAC and the bucket policy

Keep S3 Block Public Access enabled. Create `oac-config.json` and an Origin Access Control through the AWS CLI:

```json
{
  "Name": "foodierecipe-s3-oac",
  "Description": "Private S3 origin for FoodieRecipe images",
  "SigningProtocol": "sigv4",
  "SigningBehavior": "always",
  "OriginAccessControlOriginType": "s3"
}
```

```bash
aws cloudfront create-origin-access-control \
  --origin-access-control-config file://oac-config.json
```

The distribution uses the S3 REST origin, the new OAC, **Redirect HTTP to HTTPS**, only `GET`/`HEAD`, and the managed `CachingOptimized` policy. The bucket policy grants object reads only to the matching distribution:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-foodie-ai-images/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}
```

Generate an RSA key pair for CloudFront signed URLs and never commit the private key:

```bash
openssl genrsa -out cloudfront_private_key.pem 2048
openssl rsa -pubout \
  -in cloudfront_private_key.pem \
  -out cloudfront_public_key.pem
```

Upload the public key, create a trusted key group, and attach it to the image behavior. The Backend uses `CLOUDFRONT_DOMAIN`, `CLOUDFRONT_KEY_PAIR_ID`, `CLOUDFRONT_PRIVATE_KEY_BASE64`, and `CLOUDFRONT_URL_EXPIRES_IN` to generate expiring URLs.

### 3.6. Verifying URLs and caching

```bash
# A signed CloudFront URL must return 200.
curl -I "<CLOUDFRONT_SIGNED_URL>"

# Removing Signature/Key-Pair-Id or using a direct S3 URL must return 403.
curl -I "https://<DISTRIBUTION_DOMAIN>/<OBJECT_KEY>"
curl -I "https://my-foodie-ai-images.s3.ap-southeast-1.amazonaws.com/<OBJECT_KEY>"

# Request the valid URL again and inspect X-Cache/Age for edge caching.
curl -I "<CLOUDFRONT_SIGNED_URL>"
```

Object keys contain a timestamp or hash, allowing a long TTL without serving stale images. Create an invalidation only when the object key cannot change.

## 4. Knowledge and skills gained

- Understood CloudFront distributions, origins, behaviors, and edge caching.
- Protected an S3 origin with Origin Access Control.
- Designed cache keys, TTLs, and versioned object keys.
- Measured cache hit/miss behavior and image-loading time.
- Deployed RDS, EC2, Docker, Nginx, an IAM role, and Secrets Manager.
- Ran migrations and health checks and verified automatic container restart.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| CloudFront returned 403 for an existing object | Checked OAC, bucket policy, and object key |
| An old image remained cached | Used a version/hash in the key or a targeted invalidation |
| Cache hit ratio was low | Reduced query and header variation in the cache key |
| The container failed to start after deployment | Checked migrations, environment variables, and `docker logs` |
| The API worked locally but was unreachable externally | Checked Nginx, the Security Group, and loopback-only port `3001` |

## 6. Deliverables

- A CloudFront distribution reading from a private S3 bucket.
- CloudFront image URLs integrated into NestJS and Next.js.
- A cache policy and object-versioning strategy improving access speed.
- The NestJS Backend deployed on EC2, connected to RDS, and served through Nginx.

## 7. Next-week plan

Deploy the Next.js Frontend, configure CloudWatch, integrate the full pipeline, test it, and complete documentation.
