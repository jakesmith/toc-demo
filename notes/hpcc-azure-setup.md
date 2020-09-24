# Creating an Azure AKS cluster for HPCC-k8s

### Login (not needed if using Azure Cloud Shell)
`az login`

### Create a resource group
`az group create --name jcshpccdemo-rg --location eastus2`

#### If don't have a SSH key pair, need to create one [or you can pass --generate-ssh-keys]
`ssh-keygen -m PEM -t rsa -b 4096`


### Create the AKS cluster
`az aks create --resource-group jcshpccdemo-rg --name jcshpccdemo-aks --node-vm-size Standard_B4ms --node-count 1 --min-count 1 --max-count 10 --enable-cluster-autoscaler --enable-managed-identity --location eastus2`

### Configure credential access to k8s cluster
`az aks get-credentials --resource-group jcshpccdemo-rg --name jcshpccdemo-aks --admin --overwrite-existing`

#***You now have a K8s AKS cluster ready to use.***

# Manage an Azure storage account

### Create account
`az storage account create --name jcshpccdemosa --resource-group jcshpccdemo-rg --location eastus2 --sku Standard_LRS`
#### NB: default accessTier=Hot, default kind=StorageV2, default sku=Standard_RAGRS

### Retrieve access key
`az storage account keys list --account-name jcshpccdemosa --query "[0].value" -o tsv`
#### example output:
Yz6EKQP/vUAHFCCYAr1xb9DWKNV7cM1BdS0Pzvhg6IZPxS0XZzyUykS4wQ8lhFWqqv3MA0z9BAZg0ODRB4dwuQ==

### Create a storage container
`az storage container create --account-name jcshpccdemosa --account-key <storage-acccess-key> --name jcshpccdemo-container`

#### Store this key in a K8s secret yaml file:
```
apiVersion: v1
kind: Secret
metadata:
  name: jcshpccstorage
type: Opaque
stringData:
  key: <storage-access-key>
```

#### Some other useful commands
`az account list -o table` - list subscriptions (without -o table, it's more verbose yaml output)

`az account list-locations -o table` - list supported locations for your subscription

`az aks delete --resource-group jcshpccdemo-rg --name jcshpccdemo-aks --yes` - Delete the AKS cluster

`az group delete --name jcshpccdemo-rg` - delete entire resource group

`az storage container delete --account-name jcshpccdemosa --account-key <storage-acccess-key> --name jcshpccdemocontainer` - delete storage container

`az storage account delete --name jcshpccdemosa --resource-group jcshpccdemo-rg --yes` - delete storage account

