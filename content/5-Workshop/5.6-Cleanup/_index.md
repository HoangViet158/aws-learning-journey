---
title: "Clean Up Resources"
date: 2026-06-22
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Summary

You completed the FoodieRecipe flow from Next.js image upload through NestJS, S3, Rekognition, Bedrock, and CloudFront delivery. This final section removes workshop resources to prevent ongoing charges.

{{% notice danger %}}
Delete only resources created specifically for this workshop. Never delete a bucket, database, role, secret, or log group used by another environment. Back up any required data first.
{{% /notice %}}

#### 1. Stop new requests

1. Stop the `web` application and image-upload scripts.
2. Stop the NestJS container or place the API in maintenance mode.
3. Confirm that no image record remains in `processing`.

#### 2. Delete the CloudFront distribution

1. Open CloudFront and select the FoodieRecipe distribution.
2. Choose **Disable** and wait for the update to complete.
3. Choose **Delete**.
4. Delete the Origin Access Control if no other distribution uses it.

#### 3. Empty and delete the S3 image bucket

1. Delete objects under both `uploads/` and `delivery/`.
2. If versioning is enabled, delete object versions and delete markers.
3. Remove lifecycle/CORS configuration if needed.
4. Delete the image bucket after it is empty.

#### 4. Delete Backend resources

If created only for the workshop:

1. Stop and terminate the EC2 instance.
2. Delete unreferenced Security Groups.
3. Delete RDS; create a final snapshot only when data must be retained.
4. Delete unnecessary test snapshots/backups.

#### 5. Delete secrets, roles, and logs

1. Schedule deletion of `foodierecipe/database` if it is workshop-only.
2. Detach policies and delete `FoodieRecipeBackendRole` after deleting EC2.
3. Delete unused custom policies.
4. Delete `/foodierecipe/api` and workshop-only alarms/dashboards.

#### 6. Confirm cost status

1. Open **Cost Explorer** and filter by the FoodieRecipe tag/name.
2. Review S3, CloudFront, EC2, RDS, Rekognition, Bedrock, Secrets Manager, and CloudWatch.
3. Keep the Budget Alert for several days if delayed charges require monitoring.

#### Result

- Workshop resources are removed in dependency order.
- No test S3 objects, CloudFront distribution, EC2/RDS, or secret remains.
- Shared resources and retained data remain unaffected.
