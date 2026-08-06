---
title: "Prepare NestJS, EC2, and Amazon RDS"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### 1. Create Amazon RDS

1. Open **Amazon RDS → Create database**.
2. Select PostgreSQL or the engine used by the project.
3. Select a workshop-appropriate template.
4. Set the database name to `foodierecipe`.
5. Set **Public access: No**.
6. Allow the database port only from the EC2 Security Group.
7. Enable encryption and automated backup as needed.

{{% notice warning %}}
Never open the database port to `0.0.0.0/0`. Never commit an RDS password in a tracked `.env` file.
{{% /notice %}}

#### 2. Prepare the image table

Minimum schema:

```sql
CREATE TABLE recipe_images (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  recipe_id UUID,
  object_key VARCHAR(512) NOT NULL UNIQUE,
  status VARCHAR(20) NOT NULL,
  error_code VARCHAR(80),
  ai_result JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

The `status` value is limited to `pending`, `processing`, `completed`, and `failed`.

#### 3. Prepare EC2

1. Create a Linux EC2 instance suitable for the test environment.
2. Attach `FoodieRecipeBackendRole`.
3. Allow administration only from a trusted source and expose the API through Nginx.
4. Install Docker, the Docker Compose plugin, Nginx, and the CloudWatch Agent.

#### 4. Run NestJS with Docker

Reference Dockerfile:

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]
```

Start and verify:

```bash
cd api
docker build -t foodierecipe-backend .
docker run -d --name foodierecipe-api --restart unless-stopped -p 3001:3001 foodierecipe-backend
curl http://localhost:3001/health
```

#### 5. Configure Nginx

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://127.0.0.1:3001/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 2m;
    }
}
```

Reload Nginx and call `/api/health` to verify the reverse proxy.

#### 6. Read the secret in NestJS

The Backend uses the EC2 IAM role to call Secrets Manager during startup. No static access key is stored on the instance.

Expected result:

- The NestJS container runs behind Nginx.
- NestJS retrieves the database secret.
- The RDS connection succeeds.
- The health endpoint exposes no sensitive data.
