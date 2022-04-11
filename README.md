# Azure Infrastructure Operations Project: Deploying a scalable IaaS web server in Azure

## Introduction
For this project, you will write a Packer template and a Terraform template to deploy a customizable, scalable web server in Azure.

## Getting Started
1. Clone this repository

2. Create your infrastructure as code

3. Update this README to reflect how someone would use your code.

## Dependencies
1. Create an [Azure Account](https://portal.azure.com) 
2. Install the [Azure command line interface](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
3. Install [Packer](https://www.packer.io/downloads)
4. Install [Terraform](https://www.terraform.io/downloads.html)

## Instructions
### Deploy the policy

Create the policy definitition:
```
az policy definition create --name tagging-policy --mode indexed --rules tagging_policy.json
```
Assign the policy definition:
```
az policy assignment create --policy tagging-policy --name tagging-policy
```
Check the policy was assigned:
```
az policy assignment list
```

### Create a template using packer

Login to azure:
```
az login
```

Before running packer, create a resource group to contain all the resources:
```
az group create -n udacity-rg -l eastasia
```
Create a service principal to allow packer to build templates in azure:
```
az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"
```

On the machine you are running packer from, set the following environment variables using the output from the above command, along with your subscription ID:

- ARM_CLIENT_ID
- ARM_CLIENT_SECRET
- ARM_TENANT_ID
- ARM_SUBSCRIPTION_ID

1.To set the environment variables within a specific PowerShell session, use the following code. Replace the placeholders with the appropriate values for your environment.
```
$env:ARM_CLIENT_ID="<service_principal_app_id>"
$env:ARM_SUBSCRIPTION_ID="<azure_subscription_id>"
$env:ARM_TENANT_ID="<azure_subscription_tenant_id>"
$env:ARM_CLIENT_SECRET="<service_principal_password>"
```
2.Run the following PowerShell command to verify the Azure environment variables:
```
gci env:ARM_*
```
Customize the following values in [server.json](server.json):
- managed_image_resource_group_name - The name of the resource group you created in azure
- managed_image_name - The name to give to your template
- os_type - The OS type of the base image
- image_publisher - The publisher of the base image
- image_offer -  The offer of the base image
- image_sku - The SKU of the base image
- location - The region of the image
- vm_size - The size of the VM
- azure_tags:
  - environment: Environment tag, e.g. prod, dev
  - project - Project tag
  - owner - Owner tag
  - image - Image tag
- provisioners:
  - inline - The commands to execute on your template

Create the template in azure:
```
packer build server.json
```
Note: results could be similar to this
```
Build 'azure-arm' finished after 3 minutes 49 seconds.

==> Wait completed after 3 minutes 49 seconds

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

OSType: Linux
ManagedImageResourceGroupName: packer-rg
ManagedImageName: packer-poc
ManagedImageId: /subscriptions/0b30abe9-1765-462d-a169-0d22c04ce511/resourceGroups/packer-rg/providers/Microsoft.Compute/images/packer-poc
ManagedImageLocation: East Asia
```
### Provision resources using terraform
1.Run the following command to prepare
```
terraform init
```
2.Validate the files
```
terraform validate
```
3.To create an execution plan named "solution.plan"
```
terraform plan
terraform plan -out solution.plan
```
4.Create the Infrastructure ( wait some minutes):
```
terraform apply solution.plan
```
Note: results could be similar to this
```
Apply complete! Resources: 23 added, 0 changed, 0 destroyed.
```
5.Once your resources are no longer required, delete them:
```
terraform destroy
```
6.Finally, you can delete the resource group:
```
az group delete -n packer-rg
```
