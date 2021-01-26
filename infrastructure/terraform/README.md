# Terraform

Terraform will help us quickly create and provision infrastructure.

## Basic Commands

Initialize a terraform directory

```
terraform init
```

Apply the terraform scripts so that the infrastructure may be created

```
terraform apply [optional use of variables here in command]
```

Show the created resources and the state of those resources

```
terraform show
```

Destroy the resources created

```
terraform destroy
```

## Basic File Structure

```
<name of resource>
    |--- main.tf
    |--- variables.tf
    |--- outputs.tf
```

_Explanation of Structure_

* `main.tf`
    * This should call modules, locals, and data-sources to create all resources
* `variables.tf`
    * This should contain all declarations of varaibles used in `main.tf`
* `outputs.tf`
    * This contains outputs from the resources created in `main.tf`
    * These will display after we run `terrafrom apply`