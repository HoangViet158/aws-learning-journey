---
title: "Internship Self-Assessment"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During my internship at the **FIRST CLOUD AI JOURNEY (FCAJ) BOOTCAMP**, from **22 June 2026** to **14 August 2026**, I had the opportunity to learn, practise, and apply knowledge from university in a practical working and learning environment. This period brought me closer to the process of analyzing, designing, and developing a product on a cloud platform and helped me better understand the role of Cloud and AI in a modern web system.

During the Bootcamp, my team developed **FoodieRecipe**, an AI-integrated recipe-sharing application. The product enables users to manage and search for recipes, upload food images, recognize image content, and receive suggestions that support recipe creation. The complete architecture uses **Next.js** in the `web` directory for the Frontend, **NestJS** in `api` for the Backend, **Amazon RDS for PostgreSQL** for relational data, and AWS services for storage, AI processing, content delivery, security, and monitoring.

My main responsibility focused on the intelligent image-processing workflow. I researched and developed a mechanism in which the Backend issues an **S3 pre-signed URL**, enabling the browser to upload an image directly to **Amazon S3** without sending the entire file through the API. After the upload is confirmed, **Amazon Rekognition** recognizes labels and moderates image content. Accepted images and recognition results are sent to a model through **Amazon Bedrock** to suggest a dish name, description, ingredients, and tags. Users can review, edit, and confirm the AI-generated content before saving the recipe.

For image delivery, I studied the use of **Amazon CloudFront** with a private S3 origin and Origin Access Control to improve access speed, use edge caching, and prevent direct public access to S3. The workflow uses one S3 image bucket with two prefixes: `uploads/` for original images and `delivery/` for completed images. Image processing is represented by four concise states: `pending`, `processing`, `completed`, and `failed`.

I also contributed to requirement analysis, workflow definition, architecture design, bilingual Proposal and Workshop documentation, image-flow testing, and report improvement based on feedback. The complete product architecture includes **Amazon EC2, Docker, Nginx, Amazon RDS, AWS Secrets Manager, IAM, and Amazon CloudWatch** to operate the Backend, manage data, secure sensitive information, and monitor the system. However, I was not directly responsible for deploying the Backend to EC2; my contribution focused on the Next.js and NestJS integration and the S3, Rekognition, Bedrock, and CloudFront image workflow.

Through FoodieRecipe, I significantly improved my Cloud Computing knowledge, AWS SDK integration skills, technical-documentation reading, data-flow design, access-control management, and error handling. I gained a better understanding of IAM least privilege, private S3 buckets, environment-based configuration, expiring URLs, and CloudWatch monitoring. I also developed teamwork, progress management, presentation, report-writing, and problem-solving skills while integrating multiple services.

Throughout the internship, I made every effort to complete assigned tasks on schedule, proactively consulted official AWS documentation when facing difficulties, and actively discussed solutions with my mentor and team members. Although I still have much to learn, I remain willing to receive feedback and continuously improve both the product and its documentation.

To reflect objectively on my internship, I assess myself according to the following criteria:

| No. | Criterion | Description | Good | Fair | Average |
| --- | --- | --- | :---: | :---: | :---: |
| 1 | **Professional knowledge and skills** | Applied AWS Cloud, Next.js, NestJS, and AI knowledge to the FoodieRecipe image workflow | ☐ | ✅ | ☐ |
| 2 | **Learning ability** | Proactively studied S3, Rekognition, Bedrock, CloudFront, and AWS documentation | ✅ | ☐ | ☐ |
| 3 | **Proactiveness** | Took ownership of tasks, analyzed workflows, and researched solutions before consulting the mentor | ✅ | ☐ | ☐ |
| 4 | **Responsibility** | Completed assigned implementation and documentation work on schedule and consistently | ✅ | ☐ | ☐ |
| 5 | **Discipline** | Participated fully in learning sessions and team meetings and followed Bootcamp requirements | ☐ | ✅ | ☐ |
| 6 | **Growth mindset** | Accepted feedback and refined the architecture, workflow, and documentation | ✅ | ☐ | ☐ |
| 7 | **Communication** | Discussed requirements, progress, and technical issues with the mentor and team members | ☐ | ✅ | ☐ |
| 8 | **Teamwork** | Coordinated Frontend, Backend, and AWS components to maintain a consistent workflow | ✅ | ☐ | ☐ |
| 9 | **Attitude and professionalism** | Respected the mentor, supported team members, and approached the project seriously | ✅ | ☐ | ☐ |
| 10 | **Problem solving** | Analyzed upload, access-control, and AI-result issues and proposed suitable solutions | ☐ | ✅ | ☐ |
| 11 | **Project contribution** | Completed the AI image workflow and contributed to the Proposal, Workshop, and project documentation | ☐ | ✅ | ☐ |
| 12 | **Overall assessment** | Achieved the objectives and assigned scope during the FCAJ internship | ☐ | ✅ | ☐ |

## Areas for improvement

Through the internship, I identified several areas that I should continue improving to better meet future professional requirements:

- Deepen my knowledge of AWS architecture, security, IAM least privilege, scalability, and cost optimization for AI-integrated systems.
- Improve unit and integration testing for NestJS, especially for components that use the AWS SDK and external services.
- Learn queue-based asynchronous processing to make the Rekognition and Bedrock workflow more reliable as image volume increases.
- Improve log analysis, metrics monitoring, alert configuration, and incident troubleshooting with Amazon CloudWatch.
- Strengthen communication and presentation skills so that technical ideas are delivered clearly, concisely, and persuasively during team meetings and mentor reports.
- Improve time management, task breakdown, and early risk assessment for projects involving multiple integrated components.
- Continue learning Cloud Computing, DevOps, CI/CD, and Generative AI to understand the full development and operational lifecycle of a production system.

Overall, I consider my internship at the **FIRST CLOUD AI JOURNEY BOOTCAMP** a highly valuable experience. The program strengthened my AWS knowledge and enabled me to apply Cloud and AI to FoodieRecipe, develop practical working skills, and define a clearer direction for my future growth in Cloud Computing and artificial intelligence.
