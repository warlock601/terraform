# Terraform tfvars

In Terraform, `.tfvars` files (typically with a `.tfvars` extension) are used to set specific values for input variables defined in your Terraform configuration. 

They allow you to separate configuration values from your Terraform code, making it easier to manage different configurations for different environments (e.g., development, staging, production) or to store sensitive information without exposing it in your code.

Here's the purpose of `.tfvars` files:

1. **Separation of Configuration from Code**: Input variables in Terraform are meant to be configurable so that you can use the same code with different sets of values. Instead of hardcoding these values directly into your `.tf` files, you use `.tfvars` files to keep the configuration separate. This makes it easier to maintain and manage configurations for different environments.

2. **Sensitive Information**: `.tfvars` files are a common place to store sensitive information like API keys, access credentials, or secrets. These sensitive values can be kept outside the version control system, enhancing security and preventing accidental exposure of secrets in your codebase.

3. **Reusability**: By keeping configuration values in separate `.tfvars` files, you can reuse the same Terraform code with different sets of variables. This is useful for creating infrastructure for different projects or environments using a single set of Terraform modules.

4. **Collaboration**: When working in a team, each team member can have their own `.tfvars` file to set values specific to their environment or workflow. This avoids conflicts in the codebase when multiple people are working on the same Terraform project.

## Summary

Here's how you typically use `.tfvars` files

1. Define your input variables in your Terraform code (e.g., in a `variables.tf` file).

2. Create one or more `.tfvars` files, each containing specific values for those input variables.

3. When running Terraform commands (e.g., terraform apply, terraform plan), you can specify which .tfvars file(s) to use with the -var-file option:

```
terraform apply -var-file=dev.tfvars
```

By using `.tfvars` files, you can keep your Terraform code more generic and flexible while tailoring configurations to different scenarios and environments.

## Storing real secrets inside .tfvars and committing them to Git is a huge security risk. So there are 3 ways to achieve this:
- Add .tfvars files with secrets to .gitignore so they never get committed. First we put "sensitive=true" inside the variable block for any secret and then put its value inside .tfvars and put .tfvars inside .gitignore.
```hcl
# variables.tf
variable "aws_access_key" {
  type      = string
  sensitive = true      
}

variable "aws_secret_key" {
  type      = string
  sensitive = true
}
```

```hcl
# secrets.tfvars
aws_access_key = "AKIA123..."
aws_secret_key = "abcd1234..."
```
"sensitive = true" makes sure that when a specific resource is created, all these values will not be displayed. 
- Store .tfvars securely in S3. This becomes very hectic process as every time we want to do apply, we need pull the .tfvars from S3 and if we forget, it will use the default values. We you can store .tfvars in S3 bucket and pull it before running Terraform:
```sh
# Upload to S3
aws s3 cp secrets.tfvars s3://my-secure-bucket/terraform/secrets.tfvars
```

```sh
# Downliad before Terraform Apply
aws s3 cp s3://my-secure-bucket/terraform/secrets.tfvars ./secrets.tfvars
terraform apply -var-file=secrets.tfvars
```
We can also use SSE-KMS encryption inside S3 bucket to store these values securely inside S3. 

- Instead of .tfvars, use secret managers: AWS SSM Parameter Store or AWS Secrets Manager, HashiCorp Vault. If we were to have secrets in .tfvars so instead we can use these secret managers as they provide additional functionalities like expiration, secret rotation etc.
```hcl
data "aws_ssm_parameter" "db_password" {
  name = "/myapp/db_password"
  with_decryption = true
}

variable "db_password" {
  type      = string
  sensitive = true
  default   = data.aws_ssm_parameter.db_password.value
}
```
