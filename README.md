# Deploy AWS ECS Cluster

`bitovi/github-actions-deploy-ecs` Deploys a ECS Cluster.

This action uses the new GitHub Actions Commons, that is used by many Bitovi GitHub Actions, and so it's constantly evolving and improving.


## You are here
With this action, you can create your ECS (Fargate or EC2) cluster, with tasks and service definitions in a matter of minutes. 

If you would like to deploy a backend app/service, check out our other actions:
| Action | Purpose |
| ------ | ------- |
| [Deploy Docker to EC2](https://github.com/bitovi/github-actions-deploy-docker-to-ec2) | Deploys a repo with a Dockerized application to a virtual machine (EC2) on AWS |
| [Deploy static site to AWS (S3/CDN/R53](https://github.com/marketplace/actions/deploy-static-site-to-aws-s3-cdn-r53) | Hosts a static site in AWS S3 with CloudFront |

# Need help or have questions?
This project is supported by [Bitovi, A DevOps consultancy](https://www.bitovi.com/services/devops-consulting).

You can **get help or ask questions** on our:

- [Discord Community](https://discord.gg/J7ejFsZnJ4Z)

Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/services/devops-consulting).

# Basic Use

For basic usage, create `.github/workflows/deploy.yaml` with the following to build on push.
```yaml
name: Deploy ECS
on:
  push:
    branches: [ main ]

jobs:
  deploy-ecs:
    runs-on: ubuntu-latest
    steps:
    - name: Create an ECS-Fargate deploy
      uses: bitovi/github-actions-deploy-ecs@v0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID_SANDBOX }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY_SANDBOX }}
        aws_default_region: us-east-1

        aws_ecs_enable: true
        aws_ecs_task_cpu: 1024,2048
        aws_ecs_task_mem: 2048,6144
        aws_ecs_app_image: some.dkr.ecr.us-east-1.amazonaws.com/repo:fe,some.dkr.ecr.us-east-1.amazonaws.com/repo:be
        aws_ecs_assign_public_ip: true
        aws_ecs_container_port: 3000,3001
        aws_ecs_lb_redirect_enable: true
        aws_ecs_lb_container_path: 'api'
        aws_ecs_cloudwatch_enable: true
        aws_ecs_cloudwatch_retention_days: 7
        aws_ecs_additional_tags: '{\"key\":\"value\",\"key2\":\"value2\"}'

        aws_r53_enable: true
        aws_r53_domain_name: bitovi.com
        aws_r53_sub_domain_name: ecs-test
```

# Inputs

The following inputs can be used as `step.with` keys

## Input groups 
1. [AWS Specific](#aws-specific)
1. [GitHub Commons main inputs](#github-commons-main-inputs)
1. [ECS](#ecs-inputs)
1. [Secrets and Environment Variables](#secrets-and-environment-variables-inputs)
1. [VPC](#vpc-inputs)
1. [DNS](#dns-inputs)

#### **AWS Specific**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_access_key_id` | String | AWS access key ID |
| `aws_secret_access_key` | String | AWS secret access key |
| `aws_session_token` | String | AWS session token |
| `aws_default_region` | String | AWS default region. Defaults to `us-east-1` |
| `aws_resource_identifier` | String | Set to override the AWS resource identifier for the deployment. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. Use with destroy to destroy specific resources. |
| `aws_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to all provisioned resources.|
<hr/>
<br/>

#### **GitHub Commons main inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `checkout` | Boolean | Specifies if this action should checkout the code (i.e. whether or not to run the uses: actions/checkout@v3 action prior to deploying so that the deployment has access to the repo files). Defaults to `true`. |
| `bitops_code_only` | Boolean | If `true`, will run only the generation phase of BitOps, where the Terraform and Ansible code is built. |
| `bitops_code_store` | Boolean | Store BitOps generated code as a GitHub artifact. |
| `tf_stack_destroy` | Boolean  | Set to `true` to destroy the stack - Will delete the `elb logs bucket` after the destroy action runs. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. Defaults to `tf-state-aws`, `tf-state-ecr` or `tf-state-eks.` |
| `tf_state_file_name_append` | String | Appends a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. (Can co-exist with `tf_state_file_name`) |
| `tf_state_bucket` | String | AWS S3 bucket name to use for Terraform state. See [note](#s3-buckets-naming) | 
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. Default is `false`. |


#### **ECS Inputs***
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_ecs_enable`| Boolean | Toggle ECS Creation. Defaults to `false`. |
| `aws_ecs_service_name`| String | Elastic Container Service name. |
| `aws_ecs_cluster_name`| String | Elastic Container Service cluster name. |
| `aws_ecs_service_launch_type`| String | Configuration type. Could be `EC2`, `FARGATE` or `EXTERNAL`. Defaults to `FARGATE`. |
| `aws_ecs_task_type`| String | Configuration type. Could be `EC2`, `FARGATE` or empty. Will default to `aws_ecs_service_launch_type` if none defined. (Blank if `EXTERNAL`). |
| `aws_ecs_task_name`| String | Elastic Container Service task name. If task is defined with a JSON file, should be the same as the container name. |
| `aws_ecs_task_execution_role`| String | Elastic Container Service task execution role name from IAM. Defaults to `ecsTaskExecutionRole`. |
| `aws_ecs_task_json_definition_file`| String | Name of the json file containing task definition. Overrides every other input. |
| `aws_ecs_task_network_mode`| String | Network type to use in task definition. One of `none`, `bridge`, `awsvpc`, and `host`. |
| `aws_ecs_task_cpu`| String | Task CPU Amount. |
| `aws_ecs_task_mem`| String | Task Mem Amount. |
| `aws_ecs_container_cpu`| String | Container CPU Amount. |
| `aws_ecs_container_mem`| String | Container Mem Amount. |
| `aws_ecs_node_count`| String | Node count for ECS Cluster. |
| `aws_ecs_app_image`| String | Name of the container image to be used. |
| `aws_ecs_security_group_name`| String | ECS Secruity group name. |
| `aws_ecs_assign_public_ip`| Boolean | Assign public IP to node. |
| `aws_ecs_container_port`| String | Comma separated list of container ports. One for each. |
| `aws_ecs_lb_port`| String | Comma serparated list of ports exposed by the load balancer. One for each. |
| `aws_ecs_lb_redirect_enable`| String | Toggle redirect from HTTP and/or HTTPS to the main port. |
| `aws_ecs_lb_container_path`| String | Comma separated list of paths for subsequent deployed containers. Need `aws_ecs_lb_redirect_enable` to be true. eg. api. (For http://bitovi.com/api/). If you have multiple, set them to `api,monitor,prom,,` (This example is for 6 containers) |
| `aws_ecs_autoscaling_enable`| Boolean | Toggle ecs autoscaling policy. |
| `aws_ecs_autoscaling_max_nodes`| String | Max ammount of nodes to scale up to. |
| `aws_ecs_autoscaling_min_nodes`| String | Min ammount of nodes to scale down to. |
| `aws_ecs_autoscaling_max_mem`| String | Define autoscaling max mem. |
| `aws_ecs_autoscaling_max_cpu`| String | Define autoscaling max cpu. |
| `aws_ecs_cloudwatch_enable`| Boolean | Toggle cloudwatch for ECS. Default `false`. |
| `aws_ecs_cloudwatch_lg_name`| String | Log group name. Will default to `aws_identifier` if none. |
| `aws_ecs_cloudwatch_skip_destroy`| Boolean | Toggle deletion or not when destroying the stack. |
| `aws_ecs_cloudwatch_retention_days`| String | Number of days to retain logs. 0 to never expire. Defaults to `14`. |
| `aws_ecs_additional_tags`| JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to ECS provisioned resources.|
<hr/>
<br/>


#### **Secrets and Environment Variables Inputs**
| Name             | Type    | Description - Check note about [**environment variables**](#environment-variables). |
|------------------|---------|------------------------------------|
| `env_aws_secret` | String | Secret name to pull environment variables from AWS Secret Manager. |
| `env_repo` | String | `.env` file containing environment variables to be used with the app. Name defaults to `repo_env`. |
| `env_ghs` | String | `.env` file to be used with the app. This is the name of the [Github secret](https://docs.github.com/es/actions/security-guides/encrypted-secrets). |
| `env_ghv` | String | `.env` file to be used with the app. This is the name of the [Github variables](https://docs.github.com/en/actions/learn-github-actions/variables). |
<hr/>
<br/>


#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created |
| `aws_vpc_name` | String | Define a name for the VPC. Defaults to `VPC for ${aws_resource_identifier}`. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`|
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If no input, no private subnet will be created. Defaults to `<none>`. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region+<random>` value. If a list is defined, the first zone will be the one used for the EC2 instance. |
| `aws_vpc_id` | String | AWS VPC ID. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | AWS VPC Subnet ID. If none provided, will pick one. (Ideal when there's only one) |
| `aws_vpc_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to vpc provisioned resources.|
<hr/>
<br/>


#### **DNS Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_r53_enable` | Boolean | Set this to true if you wish to manage certificates through AWS Certificate Manager with Terraform. **See note**. Default is `false`. |
| `aws_r53_domain_name` | String | Define the root domain name for the application. e.g. bitovi.com'. |
| `aws_r53_sub_domain_name` | String | Define the sub-domain part of the URL. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. |
| `aws_r53_root_domain_deploy` | Boolean | Deploy application to root domain. Will create root and www records. Default is `false`. |
| `aws_r53_enable_cert` | Boolean | Set this to true if you wish to manage certificates through AWS Certificate Manager with Terraform. **See note**. Default is `false`. | 
| `aws_r53_cert_arn` | String | Define the certificate ARN to use for the application. **See note**. |
| `aws_r53_create_root_cert` | Boolean | Generates and manage the root cert for the application. **See note**. Default is `false`. |
| `aws_r53_create_sub_cert` | Boolean | Generates and manage the sub-domain certificate for the application. **See note**. Default is `false`. |
| `aws_r53_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to R53 provisioned resources.|
<hr/>
<br/>

## Contributing
We would love for you to contribute to [`bitovi/github-actions-storybook-to-github-pages`](hhttps://github.com/bitovi/github-actions-storybook-to-github-pages).   [Issues](https://github.com/bitovi/github-actions-storybook-to-github-pages/issues) and [Pull Requests](https://github.com/bitovi/github-actions-storybook-to-github-pages/pulls) are welcome!


## Note about resource identifiers

Most resources will contain the tag `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`, some of them, even the resource name after. 
We limit this to a 60 characters string because some AWS resources have a length limit and short it if needed.

We use the kubernetes style for this. For example, kubernetes -> k(# of characters)s -> k8s. And so you might see some compressions are made.

For some specific resources, we have a 32 characters limit. If the identifier length exceeds this number after compression, we remove the middle part and replace it for a hash made up from the string itself. 

## Note about tagging

There's the option to add any kind of defined tag's to each grouping module. Will be added to the commons tagging.
An example of how to set them: `{"key1": "value1", "key2": "value2"}`'

### S3 buckets naming

Buckets names can be made of up to 63 characters. If the length allows us to add -tf-state, we will do so. If not, a simple -tf will be added.

## CERTIFICATES - Only for AWS Managed domains with Route53

As a default, the application will be deployed and the ELB public URL will be displayed.

If `aws_r53_domain_name` is defined, we will look up for a certificate with the name of that domain (eg. `example.com`). We expect that certificate to contain both `example.com` and `*.example.com`. 

Setting `aws_r53_create_root_cert` to `true` will create this certificate with both `example.com` and `*.example.com` for you, and validate them. (DNS validation).

Setting `aws_r53_create_sub_cert` to `true` will create a certificate **just for the subdomain**, and validate it.

> :warning: Be very careful here! **Created certificates are fully managed by Terraform**. Therefor **they will be destroyed upon stack destruction**.

To change a certificate (root_cert, sub_cert, ARN or pre-existing root cert), you must first set the `aws_r53_enable_cert` flag to false, run the action, then set the `aws_r53_enable_cert` flag to true, add the desired settings and excecute the action again. (**This will destroy the first certificate.**)

This is necessary due to a limitation that prevents certificates from being changed while in use by certain resources.

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/bitovi/github-actions-storybook-to-github-pages/blob/main/LICENSE).

# Provided by Bitovi
[Bitovi](https://www.bitovi.com/) is a proud supporter of Open Source software.

# We want to hear from you.
Come chat with us about open source in our Bitovi community [Discord](https://discord.gg/J7ejFsZnJ4Z!