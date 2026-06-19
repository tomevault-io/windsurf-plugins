---
trigger: always_on
description: File path to save the generated YAML template
---


# CloudFormation Template Generation Skill

You are an expert AWS DevOps engineer. When this skill is invoked, you will:

1. Generate an AWS CloudFormation YAML template following the exact prompt
   structure defined in the **Generation Prompt** section below.
2. Save the template to the file path specified by `{{output}}`.
3. Call the `validate_cfn` skill on the saved file to validate it.
4. Report the outcome clearly using the **Output Format** section.

Do NOT delegate generation to another model or tool. You generate the template
yourself by following the instructions in this file.

---

## Generation Prompt

Assemble the prompt in the following order and use it to generate the template:

### System context (apply to your own reasoning)

> You are an expert in AWS CloudFormation template generation. Your task is to
> generate and improve templates based on feedback. Write your complete
> CloudFormation YAML template inside `<iac_template></iac_template>` tags.

### User message structure

Combine these three parts in order:

**TOP:**
```
You are an expert AWS DevOps engineer with extensive experience in creating
CloudFormation templates. Your task is to generate a valid, production ready
and deployable AWS CloudFormation YAML template based on the following
business need:

<business_need>
{{prompt}}
</business_need>
```

**BOTTOM (chain-of-thought instructions):**
```
Instructions:
1. Analyze the business need carefully.
2. Generate a complete CloudFormation YAML template that fulfills this need.
3. Be sure to fully specify all resources needed in the Resources section.
4. Ensure high accuracy and deployment success by following these guidelines:
   a. Start the template with 'AWSTemplateFormatVersion: 2010-09-09'.
   b. Include all necessary resources to meet the business need.
   c. Provide all required properties for each resource.
   d. Use proper YAML syntax and indentation (2-space).
   e. Follow AWS CloudFormation best practices and cfn-lint rules.
   f. End the template with the last property of the last resource.
5. Do not include any explanations, markdown formatting, or backticks in
   your output.

Before generating the final template, wrap your planning process in
<template_planning> tags. In this section:
1. List the key AWS services mentioned or implied in the business need.
2. Outline the main sections of the template (Parameters, Resources, Outputs).
3. Consider dependencies between resources and their ordering.
4. Think about any parameters or mappings needed for flexibility.
5. Consider outputs useful after stack creation.

After your planning, provide the complete CloudFormation YAML template as
your final output inside <iac_template></iac_template> tags.
```

**KNOWLEDGE INJECTION (append after BOTTOM — mandatory rules):**

```
--- CRITICAL RULES (must follow to avoid known deployment failures) ---

PARAMETERS & GREENFIELD DEPLOYMENTS:
- NEVER create Parameters for VpcId, SubnetId, SubnetIds, KeyName,
  KeyPairName, BucketName, S3BucketName, EmailAddress, or any value
  that requires user input at deploy time. This is the #1 cause of
  ValidationError deployment failures (8.3% of all errors observed).
- If VPC/Subnet is needed, CREATE them inline as AWS::EC2::VPC and
  AWS::EC2::Subnet resources. Never reference existing infrastructure.
- If EC2 KeyPair is needed, create AWS::EC2::KeyPair or use SSM Session
  Manager instead.

DEPRECATED / INVALID RUNTIMES — use these exact values:
- Lambda Python  → 'python3.12'   (python3.8 disabled 2025-02-28)
- Lambda Node.js → 'nodejs20.x'   (nodejs14.x disabled 2024-07-09)
- RDS MySQL      → one of: '8.0.32','8.0.36','8.0.39','8.0.40','8.4.3'
                   NOT '8.0' or '5.7'
- Aurora MySQL   → '8.0.mysql_aurora.3.07.0' or '8.0.mysql_aurora.3.08.0'

S3:
- Do NOT use the 'AccessControl' property; use AWS::S3::BucketPolicy.
- Set PublicAccessBlockConfiguration inside AWS::S3::Bucket — do NOT
  create 'AWS::S3::BucketPublicAccessBlock' (resource type does not exist).
- S3 event notifications go inside NotificationConfiguration in
  AWS::S3::Bucket — do NOT create 'AWS::S3::BucketNotification'.
- Always pair DeletionPolicy with UpdateReplacePolicy on stateful resources.

INTRINSIC FUNCTIONS:
- Use !Sub only when the string contains ${Variable} references.
- Do NOT use Fn::Sub on static strings (cfn-lint warns: 'Fn::Sub isn't
  needed because there are no variables').
- Always use AWSTemplateFormatVersion: '2010-09-09' (NOT '2010-09-01').

API GATEWAY:
- Valid IntegrationType values: 'AWS', 'AWS_PROXY', 'HTTP', 'HTTP_PROXY',
  'MOCK' — never lowercase ('aws', 'aws_proxy').

DYNAMODB:
- AttributeDefinitions must list ONLY attributes used in KeySchema or GSI.
- ReadCapacityUnits/WriteCapacityUnits are invalid inside GSI when
  BillingMode is PAY_PER_REQUEST.

SECURITY GROUPS:
- Omit FromPort/ToPort when IpProtocol is '-1' (all traffic).

IAM:
- Always use Version: '2012-10-17' in policy documents.
- Do NOT create AWS::IAM::RolePolicyAttachment — it does not exist; use
  inline Policies or AWS::IAM::ManagedPolicy + attach via ManagedPolicyArns.

RESOURCE TYPES THAT DO NOT EXIST (never generate):
- AWS::S3::BucketPublicAccessBlock
- AWS::S3::BucketNotification
- AWS::IAM::RolePolicyAttachment
- AWS::IAM::AccountPasswordPolicy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iksena/generate_cfn](https://github.com/iksena/generate_cfn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
