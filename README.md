# AWS App Runner Module
Terraform module which creates AWS AppRunner resources, currently only creates `aws_apprunner_service`, and have to provide arns for extra apprunner related resources.

## Future Improvements Plan 
- Support Custom VPC using [aws_apprunner_vpc_connector](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/apprunner_vpc_connector) resource.
- Support Custom autoscaling config using [aws_apprunner_auto_scaling_configuration_version](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/apprunner_auto_scaling_configuration_version) resource.
- Support custom domains using [aws_apprunner_custom_domain_association](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/apprunner_custom_domain_association) resource.
- Support creation of Github connection with [aws_apprunner_connection](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/apprunner_connection) resource.

## Usage

### The Official [AWS App Runner hello app](https://github.com/aws-containers/hello-app-runner) example that uses public ECR image source

```hcl
module "hello_app_runner" {
  source = "app.terraform.io/NewMathData/apprunner/aws"

  create = true
  service_name             = "hello-app-runner"

  tags = {
    Name = "hello-app-runner"
  }

  service_source_type      = "image"
  image_repository_type    = "ECR_PUBLIC"
  image_identifier         = "public.ecr.aws/aws-containers/hello-app-runner:latest"
  auto_deployments_enabled = false # Must set to false to disable auto deployment for ECR_PUBLIC type
}
```

### Create App runner service from private image source (ECR) for example

> Example uses [aws-app-runner-rust-example](https://github.com/bhegazy/aws-app-runner-rust-example)

```hcl
module "image_repository_private" {
  source = "app.terraform.io/NewMathData/apprunner/aws"

  create = true
  service_name             = "my-service"

  tags = {
    Name = "my-service"
  }

  service_source_type      = "image"
  auto_deployments_enabled = true
  image_repository_type    = "ECR"
  image_access_role_arn    = module.image_repository_private_ecr_role.iam_role_arn
  image_identifier         = "112233445566.dkr.ecr.us-east-1.amazonaws.com/aws-app-runner-rust-example:latest"
  image_configuration      = {
    port                          = 8080
    start_command                 = "./aws-app-runner-rust-example"
    runtime_environment_variables = {
      ENV_VAR_1 = "value1"
      ENV_VAR_2 = "value2"
    }
  }
}
```

### Create App runner service from code source that have an app config (`apprunner.yml` file).

```hcl
module "code_repository_source" {
  source = "app.terraform.io/NewMathData/apprunner/aws"
  
  create = true
  service_name             = "my-service"
  
  tags = {
    Name = "my-service"
  }
  
  service_source_type      = "code"
  auto_deployments_enabled = true
  code_connection_arn       = aws_apprunner_connection.main.arn
  code_repository_url       = "https://github.com/bhegazy/apprunner-python-app"
  code_version_type         = "BRANCH"
  code_version_value        = "main"
  code_configuration_source = "REPOSITORY"
}
```
### Examples
- [AWS App Runner Hello App](https://github.com/bhegazy/terraform-aws-apprunner/blob/main/examples/hello-app-runner)
- [Complete Code Source](https://github.com/bhegazy/terraform-aws-apprunner/tree/main/examples/complete_code_source_api)
- [Code Source with App Config (apprunner.yaml)](https://github.com/bhegazy/terraform-aws-apprunner/tree/main/examples/code_source_config_apprunner.yaml)
- [Image Source (Private ECR)](https://github.com/bhegazy/terraform-aws-apprunner/tree/main/examples/image_repository_private)

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.12.2 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 5.100.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 5.100.0 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_apprunner_service.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/apprunner_service) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_auto_deployments_enabled"></a> [auto\_deployments\_enabled](#input\_auto\_deployments\_enabled) | Whether continuous integration from the source repository is enabled for the App Runner service. Defaults to true. | `bool` | `true` | no |
| <a name="input_auto_scaling_configuration_arn"></a> [auto\_scaling\_configuration\_arn](#input\_auto\_scaling\_configuration\_arn) | The ARN of auto scaling configuration for the App Runner service | `string` | `""` | no |
| <a name="input_code_configuration_source"></a> [code\_configuration\_source](#input\_code\_configuration\_source) | The source of the App Runner configuration. Valid values: REPOSITORY, API | `string` | `"REPOSITORY"` | no |
| <a name="input_code_configuration_values"></a> [code\_configuration\_values](#input\_code\_configuration\_values) | Basic configuration for building and running the App Runner service. Supports runtime\_environment\_secrets for secret environment variables. Use this parameter to quickly launch an App Runner service without providing an apprunner.yaml file in the source code repository (or ignoring the file if it exists). | `any` | `{}` | no |
| <a name="input_code_connection_arn"></a> [code\_connection\_arn](#input\_code\_connection\_arn) | The connection ARN to use for the App Runner service if the service\_source\_type is 'code' | `string` | `""` | no |
| <a name="input_code_repository_url"></a> [code\_repository\_url](#input\_code\_repository\_url) | The location of the repository that contains the source code. This is required for service\_source\_type 'code' | `string` | `""` | no |
| <a name="input_code_version_type"></a> [code\_version\_type](#input\_code\_version\_type) | The type of version identifier. For a git-based repository, branches represent versions. Valid values: BRANCH | `string` | `"BRANCH"` | no |
| <a name="input_code_version_value"></a> [code\_version\_value](#input\_code\_version\_value) | A source code version. For a git-based repository, a branch name maps to a specific version. App Runner uses the most recent commit to the branch. | `string` | `"main"` | no |
| <a name="input_create"></a> [create](#input\_create) | Controls if App Runner resources should be created | `bool` | `true` | no |
| <a name="input_health_check_configuration"></a> [health\_check\_configuration](#input\_health\_check\_configuration) | The health check configuration for the App Runner service | `map(string)` | `{}` | no |
| <a name="input_image_access_role_arn"></a> [image\_access\_role\_arn](#input\_image\_access\_role\_arn) | The access role ARN to use for the App Runner service if the service\_source\_type is 'image' and image\_repository\_type is not 'ECR\_PUBLIC' | `string` | `""` | no |
| <a name="input_image_configuration"></a> [image\_configuration](#input\_image\_configuration) | Configuration for running the identified image. Supports runtime\_environment\_secrets for secret environment variables. | `any` | `{}` | no |
| <a name="input_image_identifier"></a> [image\_identifier](#input\_image\_identifier) | The identifier of an image. For an image in Amazon Elastic Container Registry (Amazon ECR), this is an image name. | `string` | `""` | no |
| <a name="input_image_repository_type"></a> [image\_repository\_type](#input\_image\_repository\_type) | The type of the image repository. This reflects the repository provider and whether the repository is private or public. Defaults to ECR | `string` | `"ECR"` | no |
| <a name="input_instance_configuration"></a> [instance\_configuration](#input\_instance\_configuration) | The instance configuration for the App Runner service | `map(string)` | `{}` | no |
| <a name="input_ip_address_type"></a> [ip\_address\_type](#input\_ip\_address\_type) | App Runner provides you with the option to choose between Internet Protocol version 4 (IPv4) and dual stack (IPv4 and IPv6) for your incoming public network configuration. Valid values: IPV4, DUAL\_STACK. Default: IPV4. | `string` | `"IPV4"` | no |
| <a name="input_is_publicly_accessible"></a> [is\_publicly\_accessible](#input\_is\_publicly\_accessible) | Specifies whether your App Runner service is publicly accessible. To make the service publicly accessible set it to true. To make the service privately accessible, from only within an Amazon VPC set it to false. | `bool` | `true` | no |
| <a name="input_kms_key_arn"></a> [kms\_key\_arn](#input\_kms\_key\_arn) | The ARN of the custom KMS key to be used to encrypt the copy of source repository and service logs. By default, App Runner uses an AWS managed CMK | `string` | `""` | no |
| <a name="input_observability_configuration_arn"></a> [observability\_configuration\_arn](#input\_observability\_configuration\_arn) | ARN of the observability configuration that is associated with the service. Specified only when observability\_enabled is true. | `string` | `""` | no |
| <a name="input_observability_enabled"></a> [observability\_enabled](#input\_observability\_enabled) | When true, an observability configuration resource is associated with the service. | `bool` | `false` | no |
| <a name="input_service_name"></a> [service\_name](#input\_service\_name) | App Runner service name | `string` | `""` | no |
| <a name="input_service_source_type"></a> [service\_source\_type](#input\_service\_source\_type) | The service source type, valid values are 'code' or 'image' | `string` | `"image"` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | A map of tags to add to all resources | `map(string)` | `{}` | no |
| <a name="input_vpc_connector_arn"></a> [vpc\_connector\_arn](#input\_vpc\_connector\_arn) | The ARN of the VPC connector to use for the App Runner service | `string` | `""` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_service_arn"></a> [service\_arn](#output\_service\_arn) | The App Runner Service ARN |
| <a name="output_service_status"></a> [service\_status](#output\_service\_status) | The App Runner Service Status |
| <a name="output_service_url"></a> [service\_url](#output\_service\_url) | The App Runner Service URL |
<!-- END_TF_DOCS -->

## Authors

Module is maintained by [Bill Hegazy](https://github.com/bhegazy).

## License

Apache 2 Licensed. See [LICENSE](https://github.com/bhegazy/terraform-aws-apprunner/tree/main/LICENSE) for full details

### Example: Using Observability and Secret Environment Variables

```hcl
module "my_app_runner" {
  source = "app.terraform.io/NewMathData/apprunner/aws"
  create = true
  service_name = "my-service"
  service_source_type = "image"
  image_repository_type = "ECR"
  image_identifier = "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest"
  image_access_role_arn = aws_iam_role.ecr_access.arn
  image_configuration = {
    port = 8080
    runtime_environment_variables = {
      ENV_VAR_1 = "value1"
    }
    runtime_environment_secrets = {
      SECRET_KEY = aws_secretsmanager_secret.example.arn
    }
  }
  observability_enabled = true
  observability_configuration_arn = aws_apprunner_observability_configuration.example.arn
}
```
