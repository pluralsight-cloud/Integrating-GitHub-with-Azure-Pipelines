Create a resource group
```
az group create --name <USERNAME>-rg --location eastus
```
# Create a container registry
```
az acr create --resource-group <RESOURCE_GROUP> --name <REGISTRY_NAME> --sku Basic
```

# Create a Kubernetes cluster
```
az aks create \
    --resource-group <RESOURCE_GROUP> \
    --name <NAME> \
    --node-count 1 \
    --enable-addons monitoring \
    --generate-ssh-keys
```