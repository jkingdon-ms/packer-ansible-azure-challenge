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
Open port 80 on the VM and view the Apache default webpage in browser.

```az vm open-port --resource-group managed-image-rg --name my-vm --port 80 && open http://$(az vm show -d -g managed-image-rg -n my-vm --query publicIps -o tsv)/```
