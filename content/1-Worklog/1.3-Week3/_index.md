---
title: "Week 3: Building the NestJS Backend"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

## 1. Objectives

- Build the image-processing module in the NestJS Backend.
- Integrate the AWS SDK for JavaScript with Amazon S3.
- Validate image format, size, and ownership.
- Build a direct-upload flow with pre-signed URLs.
- Build JWT authentication and APIs for users, recipes, ingredients, categories, likes, and comments.
- Prepare PostgreSQL migrations, a health endpoint, and a production Dockerfile.

## 2. Work plan

> **Week 3 schedule:** Monday, 06/07/2026 – Friday, 10/07/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Design the image module, controller, service, and DTOs | Establish a clear Backend structure |
| Tuesday | Integrate the S3 client and server-side upload API | Upload images under the correct object key |
| Wednesday | Add file type, size, and metadata validation | Reject invalid files before storage |
| Thursday | Build pre-signed URL and upload-confirmation APIs | Keep image bytes out of the Backend path |
| Friday | Complete auth, recipe, like/comment APIs, migrations, health checks, and the Dockerfile | Make the Backend ready for integration and deployment |

## 3. Work completed

### 3.1. NestJS image module

- Separated `ImageModule`, `ImageController`, and `ImageService`.
- Used DTOs and `ValidationPipe` for request validation.
- Generated object keys from the user, recipe, and a UUID.
- Used only four states: `pending`, `processing`, `completed`, and `failed`.
- Built `AuthModule`, `UsersModule`, `RecipesModule`, `IngredientsModule`, `CategoriesModule`, `LikesModule`, and `CommentsModule`.
- Added JWT/cookie authentication, ownership checks, pagination, and recipe search.

**Swagger displays the FoodieRecipe API documentation and endpoints:**

![FoodieRecipe API overview in Swagger](/images/1-Worklog/1.3-Week3/swagger-overview.png)

**Enter credentials to test the login API directly in Swagger:**

![Submitting login credentials through Swagger](/images/1-Worklog/1.3-Week3/swagger-login-request.png)

**The login API processes the request successfully and returns user information:**

![Successful login test result in Swagger](/images/1-Worklog/1.3-Week3/swagger-login-success.png)

### 3.2. Amazon S3 integration

- Configured the S3 client with the Region and environment-based credentials.
- Sent the correct `Content-Type` and metadata when creating objects.
- Granted only the required S3 actions to the Backend.
- Standardized errors for access denial, missing objects, and interrupted uploads.

### 3.3. Pre-signed URLs

The Backend creates an expiring upload URL bound to an object key. After the browser uploads directly to S3, the Frontend calls a confirmation API; the Backend checks the object before starting AI analysis.

Completed PostgreSQL migrations for business data, likes, and comments, along with the `/health` endpoint and a multi-stage Dockerfile. Ran unit tests, migrations, and the container locally to confirm that the Backend was ready for EC2 deployment.

**Inspect PostgreSQL data and tables after running Prisma migrations:**

![Prisma migration result inspected in Prisma Studio](/images/1-Worklog/1.3-Week3/prisma-migration-result.png)

**Inspect the PostgreSQL container with `docker ps`:**

![PostgreSQL container running with a healthy status](/images/1-Worklog/1.3-Week3/docker-postgres-running.png)

**Test the NestJS root endpoint with `curl http://localhost:3001/api`:**

![NestJS API successfully returns Hello World](/images/1-Worklog/1.3-Week3/api-hello-world.png)

## 4. Knowledge and skills gained

- Organized NestJS modules, dependency injection, and validation.
- Used the AWS SDK with S3 commands and pre-signed URLs.
- Designed a secure, stateful image-upload API.
- Handled errors and cleaned up incomplete objects.
- Designed business APIs, migrations, and a production NestJS Docker image.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| A file could spoof its MIME type | Checked MIME type, extension, and file signature |
| The URL expired during upload | Allowed a new URL while keeping the same image record |
| Upload completed without confirmation | Kept a `pending` state and cleaned up expired objects |

## 6. Deliverables

- A NestJS image module integrated with Amazon S3.
- Server-side upload and pre-signed URL flows.
- Validation, image states, and deletion handling.
- Authentication, user, recipe, ingredient, and category APIs, migrations, and a production Dockerfile.

## 7. Next-week plan

Build the complete Next.js account, recipe, like, comment, search, and image-upload interfaces and integrate them with NestJS APIs.

## 8. References

- [File upload techniques in NestJS](https://docs.nestjs.com/techniques/file-upload)
- [Configuration in NestJS](https://docs.nestjs.com/techniques/configuration)
- [Authentication in NestJS](https://docs.nestjs.com/security/authentication)
- [Generating OpenAPI/Swagger documentation with NestJS](https://docs.nestjs.com/openapi/introduction)
- [Amazon S3 examples using AWS SDK for JavaScript v3](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_s3_code_examples.html)
