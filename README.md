# packer-ansible-azure-challenge

# Steps

## Build Image
1. Create the Service Principal that has permission to build the image, and create/modify VMs

- `az ad sp create-for-rbac --name packer-service-principal --role contributor --scopes /subscriptions/{{SUBSCRIPTION_ID}}`

2. Update the `ubuntu.pkr.hcl` file with the Service Principal data

```
  client_id                         = "{{APP_ID}}"
  client_secret                     = "{{PASSWORD}}"
  subscription_id                   = "{{SUBSCRIPTION_ID}}"
  tenant_id                         = "{{TENANT_ID}}"  
```

3. Build the image 

- `packer build ubuntu.pkr.hcl`


## Deploy Image

`az vm create --resource-group managed-image-rg --name MyVM --image MyPackerImage --admin-username azureuser --generate-ssh-keys`

## Test

`az vm open-port --resource-group managed-image-rg --name MyVM --port 80`

`open http://{{VM_IP_ADDRESS}}/`
