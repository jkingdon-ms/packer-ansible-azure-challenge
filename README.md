# Build Golden Image with Packer + Ansible + Azure

# Install
- [Packer CLI](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli )
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

# How To
## Initialize Azure Resources
1. Login to Azure CLI

    ```az login```

1. Create a resource group for the image

    ```az group create --name managed-image-rg --location eastus```

1. Create the Service Principal that has permission to build the image, and create/modify VMs

    ```az ad sp create-for-rbac --name packer-service-principal --role contributor --scopes /subscriptions/{{SUBSCRIPTION_ID}}```

1. Set the Azure Credentials as environment variables from the output of the previous command

    ```
    export PKR_VAR_client_id={{APP_ID}}
    export PKR_VAR_client_secret={{PASSWORD}}
    export PKR_VAR_subscription_id={{SUBSCRIPTION_ID}}
    export PKR_VAR_tenant_id={{TENANT_ID}}
    ```

## Build Image

```packer build custom-image.pkr.hcl```

## Deploy Image
Create an Azure VM based on our custom image.

```az vm create --resource-group managed-image-rg --name my-vm --image my-custom-image --admin-username azureuser --generate-ssh-keys```

## Test
Start the VM, open port 80, and view the Apache default webpage in browser.

```az vm start -g managed-image-rg -n my-vm && az vm open-port --resource-group managed-image-rg --name my-vm --port 80 && open http://$(az vm show -d -g managed-image-rg -n my-vm --query publicIps -o tsv)/```

# GitHub Actions

## Build and Deploy
This workflow will build the image with Packer and deploy a new VM to Azure with the custom image. It runs on pushes to PRs and to the main branch. Only pushes to the main branch will trigger a VM deployment. Pushes to a PR will simply build the image and upload it to Azure. The workflow creates a new resource group per branch, in order to isolate the resources of that branch. The resource group name is the branch name in this case.

## Cleanup Branch Resource Group
This workflow will delete the resource group created by the previous workflow. It runs on branch deletion.

## Use cases:

### PR Flow
1. Developer creates a feature branch
1. Developer makes changes to image (e.g. Ansible playbook) and opens a PR
1. The workflow will build the image and upload it to Azure in the branch rg
1. The developer can then test the image by deploying a VM from the image
1. After testing, the developer can merge the PR to main
1. Upon successful build in main, the feature branch can be deleted
1. Upon feature branch deletion, the branch rg will be deleted

### Main Flow
1. Developer merges PR into main
1. The workflow will build the image and upload it to Azure in the main rg
1. The workflow will deploy a VM from the image in the main rg
