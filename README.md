# Infrastructure — ECS CI/CD

CloudFormation templates for the ECS CI/CD project, managed via GitSync.

## Stack Deployment Order

Deploy stacks in dependency order:

```
1. vpc-stack        → templates/vpc.yaml
2. security-stack   → templates/security.yaml
3. ecr-stack        → templates/ecr.yaml
4. ecs-stack        → templates/ecs.yaml
5. pipeline-stack   → templates/pipeline.yaml
```

## GitSync Setup

For each stack, connect a CloudFormation stack to this repository using the
deployment file in `deployment-files/`:

```
aws cloudformation create-stack-set \
  --stack-set-name ecs-cicd-vpc \
  --template-body file://templates/vpc.yaml
```

Or use the AWS Console:
1. CloudFormation → Create Stack → With GitSync
2. Select this repository and branch
3. Point to the relevant `deployment-files/<stack>.yaml`

## Parameters to Update

Before deploying `ecr-stack` and `pipeline-stack`, edit their deployment files:

- `deployment-files/ecr-stack.yaml` → `GitHubOrg`, `GitHubRepo`
- `deployment-files/pipeline-stack.yaml` → `GitHubOrg`, `GitHubAppRepo`

## Post-Deployment Steps

1. **Authorise the GitHub Connection** (pipeline-stack output `GitHubConnectionArn`):
   AWS Console → Developer Tools → Connections → Pending → Approve

2. **Push the first image** from the application repo to trigger the pipeline.

## Architecture

```
Internet
   │
   ▼
[ALB] (public subnets, port 80/8080)
   │
   ▼ (Security Group: ALB→ECS only)
[ECS Fargate Tasks] (private subnets)
   │
   ├─ ECR (via VPC Interface Endpoint)
   └─ CloudWatch Logs (via VPC Interface Endpoint)
```
