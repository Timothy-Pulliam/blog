---
layout: "../../layouts/BlogPost.astro"
title: "Spin Up an AWS Fargate Cluster with Cloudformation"
description: "Spin Up an AWS Fargate Cluster with Cloudformation"
pubDate: "Oct 31 2021"
heroImage: "/aws-hero.jpg"
previewText: "Spinning up an AWS Elastic Container Service Fargate Cluster (try saying that three times and fast) is a lot easier than you think. The Fargate launch type abstracts away a lot of the hardware details and the process can be automated very easily. This is where Cloudformation comes in."
---

# Overview

Spinning up an AWS Elastic Container Service Fargate Cluster (try saying that three times and fast) is a lot easier than you think. The Fargate launch type abstracts away a lot of the hardware details and the process can be automated very easily. This is where Cloudformation comes in.

Cloudformation is very useful for creating and deploying AWS resources such as EC2 instances, S3 buckets, and ECS clusters. It not only allows you to keep your infrastructure as code, but it also keeps track of all of the resources together as a logical unit known as a Stack. It goes like this.

1. Write Template File (YAML) containing AWS resources

2. Test the deployment with Taskcat by doing a dry run
3. Your resources are deployed together as a single unit known as a Stack.

The benefit of Stacks is that all of your resources are created at the same time, but also they will also be deleted at the same time when you inevitably must shut down your application. Instead of removing each resource one by one (deleting the EC2 instance, deleting the S3 bucket, deleting the security group, etc.) you run a single command `aws cloudformation delete-stack --stack-name test-stack` and everything is deleted at the same time.

# The Template File

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "AWS CloudFormation Template used to create ECS Fargte Cluster."
Parameters:
  ECSClusterName:
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9\-]*'
    ConstraintDescription:
      must begin with a letter and contain only alphanumeric
      characters, or hyphens.
    Default: "test-cluster"
    Description: The ECS cluster name
    MaxLength: "64"
    MinLength: "1"
    Type: String
Resources:
  ECSCluster:
    Type: "AWS::ECS::Cluster"
    Properties:
      ClusterName: !Sub ${ECSClusterName}
      CapacityProviders:
        - FARGATE
        - FARGATE_SPOT
      DefaultCapacityProviderStrategy:
        - CapacityProvider: FARGATE
          Weight: 1
        - CapacityProvider: FARGATE_SPOT
          Weight: 1
```

A Cloudformation template has the following main sections

- AWSTemplateFormatVersion: Metadata informing Cloudformation about what version of cloudformation template file it can expect to see.
- Description: A description of the template file
- Parameters: These are basically variables that hold values that can be substituted later in the template
- Resources: This is where we define AWS resources like EC2 instances, S3 buckets, ECS Clusters, etc.

There is also an [Outputs section](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html) you can use to capture output from created resources (ARNs, IDs, created Hostnames/URLs, etc.), but for the purposes of this template file, it is not necessary.

The resource we are creating is an [AWS::ECS::Cluster type](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-cluster.html). It uses a default capacity provider strategy. Basically, any deployment made to it will be evenly split between FARGATE and [FARGATE_SPOT instance types](https://aws.amazon.com/blogs/aws/aws-fargate-spot-now-generally-available/). That is what the weights mean. For every instance deployed via the FARGATE capacity provider, one should also be deployed via the FARGATE_SPOT capacity provider (1:1 ratio). If you want to only use FARGATE_SPOT for example, you can set the FARGATE weight to zero. For more information on CapacityProviderStrategy and weights, I invite you to read the [following documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ecs-cluster-capacityproviderstrategyitem.html#cfn-ecs-cluster-capacityproviderstrategyitem-weight).

```yaml
DefaultCapacityProviderStrategy:
  - CapacityProvider: FARGATE
    Weight: 1
  - CapacityProvider: FARGATE_SPOT
    Weight: 1
```

When you are using FARGATE_SPOT, you are not using dedicated hardware, but using cluster resources as they are made available to you. This turns out to be cheaper, although less reliable because your spot instances can be taken from you at any time. Although you can design around this using Fault-Tolerant, or parallelizable workloads.

# Testing The Template File with Taskcat

[Taskcat](https://github.com/aws-quickstart/taskcat) is a very useful tool for doing a dry run of your stack before actually deploying. It is important to test your Template files because some resources need to be created before others (e.g. an ECS Service Definition that depends upon a load balancer to be available). Also, some resources or AWS services may not be available in all regions. Taskcat will catch these errors and generate a report.

You can install Taskcat with pip. It is also a good idea to install the cloudformation linter `cfn-lint` as well. This can be used to inform you if you have any syntax errors in your template file.

`pip install taskcat cfn-lint`

Taskcat expects your directory structure to be like the following

```
ecs/
| - FargateCluster.yaml
| - .taskcat.yml
```

Note: your .taskcat.yml file must have the extension .yml and not .yaml.

The contents of `.taskcat.yml`

```yaml
project:
  name: ecs
  regions:
    - us-east-2
tests:
  default:
    template: FargateCluster.yaml
    parameters:
      ECSClusterName: "test-cluster"
```

The project name must be the same as the project directory, in this case `ecs`, which must also be all lower case (taskcat is very picky). We specify which regions we want to test in the project regions section. It is important to note that the stacks we are creating are region-specific. The rest is pretty straightforward.

Before we do our test run, we should lint the template file to make sure it is free of syntax errors which would prevent it from running. You can either use `cfn-lint` or taskcat, both do the same thing (although, I’m pretty sure taskcat just uses `cfn-lint` so you’re better off just using that).

![cfn-lint output](/fargate-cloudformation/taskcat.webp)_no output from cfn-lint means no syntax errors._

Next, we’ll do a dry run with `taskcat test run` This causes taskcat to create a temporary stack using the Template file provided in the specified regions and with the passed parameters. Afterward, it will delete the stack and report CREATE_COMPLETE if the stack was created successfully, or CREATE_FAILED if the stack failed along with any errors indicating the cause.
Next, we’ll do a dry run with `taskcat test run` This causes taskcat to create a temporary stack using the Template file provided in the specified regions and with the passed parameters. Afterward, it will delete the stack and report CREATE_COMPLETE if the stack was created successfully, or CREATE_FAILED if the stack failed along with any errors indicating the cause.

![taskcat output](/fargate-cloudformation/taskcat_test_run.webp)

Taskcat will generate an HTML file in the project directory. For example `ecs/taskcat_outputs/index.html`

![taskcat html output](/fargate-cloudformation/taskcat_html.webp)

Once the stack has passed the dry run, we can move on to deployment.

## Creating the Cloudformation Stack

Deploying the stack can be done through the AWS CLI.

`aws cloudformation create-stack --region us-east-2 --stack-name test-stack --template-body file://FargateCluster.yaml`

You can list your created stacks

`aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE`

If your stack is deployed, you should be able to see your Fargate cluster in the AWS console.

![cloudformation stacks](/fargate-cloudformation/cloudformation_stack.webp)_Successfully Created ECS Cluster_

Lastly, if you want to delete your test stack, which will delete the cluster.

`aws cloudformation delete-stack --region us-east-2 --stack-name test-stack`

## Cleaning up Taskcat Files

Okay, there is one more thing that I should mention. It is a slight nuisance on the part of taskcat. Every time you run taskcat, it creates an S3 bucket and uploads all Template files in the project directory to this bucket. It needs to do this to be able to create the stack.

![taskcat buckets](/fargate-cloudformation/taskcat_buckets.webp)_All Template files will be uploaded to these buckets_

The problem is, once you start editing larger Template files, you will run taskcat a lot, and you will get a bucket for each version of the template file you ran. It’s very annoying. So if you want to just delete all of these buckets, here is a small script that does just that.

```python
import boto3
from botocore.exceptions import ClientError
import re

s3 = boto3.resource('s3')
try:
    client = boto3.client('s3')
except ClientError as e:
    # Couldn't connect to AWS
    print("AWS API couldn't connect. exiting.")
    sys.exit(1)

r = re.compile('^tcat.*$')
task_cat_buckets = [b['Name'] for b in client.list_buckets()['Buckets'] if r.match(b['Name'])]
for bucket in task_cat_buckets:
    # must first delete contents of bucket before we can delete bucket
    s3.Bucket(bucket).objects.all().delete()
    s3.Bucket(bucket).delete()
```
