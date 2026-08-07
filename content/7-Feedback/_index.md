---
title: "Sharing and Feedback"
date: 2026-06-22
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

## Overall assessment of the internship at the First Cloud AI Journey (FCAJ) Bootcamp

### 1. Working and learning environment

During my participation in the **First Cloud AI Journey (FCAJ) Bootcamp** from **22 June 2026** to **14 August 2026**, I found it to be a professional, friendly, and highly practical learning environment. Although the program follows a Bootcamp model rather than a traditional company structure, its learning process, progress reporting, teamwork, and project development closely reflect a real working environment.

Mentors, Teaching Assistants (TAs), Team Admins, and learners were always willing to support one another. While developing **FoodieRecipe**, I could discuss architecture, AWS integration, image-processing workflows, and technical documentation with the community. This open atmosphere encouraged me to ask questions, share approaches, and learn from different perspectives.

What impressed me most was the program's emphasis on researching a problem before asking for assistance. When I encountered issues involving image uploads, S3 permissions, or AI service calls, I was guided to inspect logs, verify configuration, and consult official documentation instead of simply receiving an answer. This helped me develop a more systematic and independent problem-solving habit.

### 2. Support from mentors, Teaching Assistants, and Team Admins

Throughout the internship, I received dedicated guidance from mentors and TAs. When the team encountered difficulties defining the workflow or selecting suitable services for FoodieRecipe, the mentors helped us analyze requirements, asked guiding questions, and provided feedback so that we could refine the solution ourselves.

Their feedback on Amazon S3, Amazon Rekognition, Amazon Bedrock, and Amazon CloudFront helped me understand that a complete AI feature involves more than invoking a model. It must also address access control, data formats, processing states, failure handling, output quality, user confirmation, and operating cost.

The mentors also shared practical experience in presenting architectures, managing secrets, applying least-privilege IAM, and monitoring systems with CloudWatch. These insights helped me understand the responsibilities involved in building a Cloud system rather than focusing only on feature completion.

Team Admins also provided timely support with learning materials, schedules, events, and program-related questions. The coordination among mentors, TAs, and Team Admins allowed me to focus on learning, project work, and report completion.

### 3. Relevance to my academic major

I believe the FCAJ internship content was highly relevant to my Information Technology studies. Foundational knowledge of web development, computer networks, databases, and systems analysis was directly applied while building FoodieRecipe.

The project uses **Next.js** in the `web` directory for a Frontend supporting recipe management, likes, and comments, and **NestJS** in `api` for the Backend. It uses **Amazon RDS for PostgreSQL** for relational data, **Amazon S3** for image storage, **Amazon Rekognition** for image recognition and moderation, **Amazon Bedrock** for recipe-content suggestions, and **Amazon CloudFront** for faster image delivery.

I deployed the complete product architecture with Amazon EC2, Docker, Nginx, Amazon RDS, AWS Secrets Manager, IAM, and Amazon CloudWatch for operation, security, and monitoring. The work covered `web` and `api` development, Backend deployment to EC2, RDS connectivity, Frontend publishing through S3/CloudFront, and the complete S3–Rekognition–Bedrock–CloudFront image workflow.

Combining programming knowledge with Cloud and Generative AI helped me better understand the relationship between university subjects and the requirements of a real software product. These practical skills complemented areas that I had not previously had the opportunity to study in depth.

### 4. Learning and skill-development opportunities

The Bootcamp gave me many opportunities to develop both technical knowledge and professional skills. Through FoodieRecipe, I gained a clearer understanding of the process from requirement analysis and architecture design to feature development, testing, documentation, and result reporting.

Technically, I learned how to create S3 pre-signed URLs for direct uploads, organize objects under `uploads/` and `delivery/`, control access with IAM, use Rekognition for moderation and recognition, send data to Bedrock, and deliver content through CloudFront. I also learned to manage the four image states—`pending`, `processing`, `completed`, and `failed`—to make the workflow easier to track.

Professionally, I practised planning, task allocation, progress tracking, and team coordination. When errors occurred, I learned to inspect messages, verify each workflow step, consult technical documentation, and identify the cause before proposing a solution.

My ability to read English documentation, write Worklogs, prepare the Proposal, create a bilingual Workshop, draw architecture diagrams, and present project outcomes also improved significantly. These skills provide an important foundation for adapting to a professional working environment after graduation.

### 5. Knowledge-sharing culture and teamwork

One of the aspects I valued most at FCAJ was its knowledge-sharing culture. Members willingly shared errors they encountered, useful resources, and practical experience. This helped each person understand not only their assigned work but also other components of the complete system.

During the team project, each member was responsible for assigned tasks while collaborating on the architecture, workflow, and integration among the Frontend, Backend, and AWS services. When a member encountered difficulties, the team discussed possible solutions instead of treating each task as an isolated component.

Mentors encouraged everyone to present, discuss, and challenge ideas. This made me feel respected, strengthened my sense of responsibility, and motivated me to improve my contribution. Although the internship period was limited, I clearly experienced the professionalism, cooperation, and learning spirit of the FCAJ community.

## Personal reflection after the internship

After participating in the First Cloud AI Journey Bootcamp, I consider it a meaningful experience for both my education and career direction. What satisfied me most was applying knowledge to FoodieRecipe—a product combining web development, Cloud, and AI—instead of learning each service in isolation.

The project taught me that a real feature requires several coordinated concerns: data must be stored securely, access must be limited, AI results must be validated, users must be able to edit suggestions, and failures must be observable. These lessons gave me a more complete perspective on software development.

In addition to technical knowledge, I improved my teamwork, progress management, documentation, presentation, and feedback-reception skills. These experiences have made me more confident as I prepare to enter a professional working environment after graduation.

## Suggestions and feedback

In my view, the program is well structured, with a clear learning path and active support from mentors, TAs, and Team Admins. I would like to offer several suggestions for further improvement:

- Organize more sessions with practising Cloud Engineers, DevOps Engineers, Solution Architects, and AI Engineers.
- Add a mid-project architecture review to identify inconsistencies among the Proposal, source code, and Workshop early.
- Provide more practical examples of least-privilege IAM, CloudWatch Logs, AWS SDK error handling, and AI service cost optimization.
- Add guidance on unit testing, integration testing, and mocking AWS services so learners can test without incurring significant cost.
- Provide a checklist for each phase covering objectives, required resources, estimated cost, completion criteria, and cleanup steps.
- Create more opportunities for teams to present their products and compare different architectural approaches.

I would recommend the FCAJ Bootcamp to friends and students interested in Cloud Computing, DevOps, or AI because it provides practical content, an active support community, and opportunities to develop both technical and professional skills.

In the future, I hope to participate in advanced FCAJ programs covering AWS Solution Architecture, DevOps, Cloud security, or Generative AI. I also plan to improve FoodieRecipe with asynchronous processing, stronger testing, cost tracking, and better system observability.

Finally, I would like to express my sincere gratitude to the mentors, Teaching Assistants, Team Admins, and organizers of the First Cloud AI Journey Bootcamp for their guidance, support, and for creating a professional learning environment. My FCAJ internship helped me grow in technical knowledge, professional skills, and problem-solving ability. It will serve as an important foundation for my future development in Cloud Computing and artificial intelligence.
