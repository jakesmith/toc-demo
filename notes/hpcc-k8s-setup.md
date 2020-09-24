# Installing HPCC on K8s with Helm

### Install the repository for HPCC
`helm repo add hpcc https://hpcc-systems.github.io/helm-chart/`

#### Install HPCC Helm chart with temporary azurefile Persistent Volume's
`helm install mycluster hpcc/hpcc --set global.image.version=7.12.0-rc1 --set storage.dllStorage.storageClass=azurefile --set storage.daliStorage.storageClass=azurefile --set storage.dataStorage.storageClass=azurefile`

*For docker versions go to: [https://hub.docker.com/r/hpccsystems/platform-core/tags]*

*Reasons not to use tag 'latest': [https://vsupalov.com/docker-latest-tag/#:~:text=The%20Kubernetes%20docs%20are%20pretty,and%20hard%20to%20roll%20back]*

*NB: can use --wait to stop helm returning until fully up...*

### Can watch state changes of pods with
`kubectl get pods --watch`

### Get default/base HPCC helm chart configuration
`helm show values hpcc/hpcc > myvalues.yaml`

## Persisting data

### Persisting data using a K8s Persistent Volumes
There are multiple ways to persist data.
In the 1st installation above, we set the storageClass, which defaults to using deletable "azurefile" PersistVolume's in Azure.
When the pods are no longer using the volumes, they are are automatically deleted.
i.e. when the hpcc helm chart is removed, the storage disappears.


We can install persistent "azurefile" storage using the helm chart in helm/examples/azure/hpcc-azurefile [https://github.com/hpcc-systems/HPCC-Platform/tree/master/helm/examples/azure]:
`helm install azstorage hpcc/hpcc-azurefile | fgrep -A 100 "storage:" > azurefile.yaml`

NB: we capture the output from the helm chart installation, because it generates the PVC names we'll use when installing the hpcc helm chart.

Now install the hpcc chart using the captured output from above:

`helm install mycluster hpcc/hpcc --set global.image.version=7.12.0-rc1 --values myvalues.yaml --values azurefile.yaml`


### Persisting data using Blob storage

#### Create storage account
`az storage account create --name jcshpccdemosa --resource-group jcshpccdemo-rg --location eastus2 --sku Standard_LRS`

*NB: default accessTier=Hot, default kind=StorageV2, default sku=Standard_RAGRS*

*Storage SKU types: [https://docs.microsoft.com/en-us/rest/api/storagerp/srp_sku_types]*

*Refer to Gavin Halliday's blog re. a storage plane setup utilizing Azure Blobs: [https://hpccsystems.com/blog/persisting-data-cloud2]*


#### Some other useful commands
`helm list` - list installed helm charts

`helm uninstall mycluster` - delete the install hpcc chart

`helm uninstall azstorage` - delete the installed azure storage chart

`kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name` - see which VM's are running which pods
