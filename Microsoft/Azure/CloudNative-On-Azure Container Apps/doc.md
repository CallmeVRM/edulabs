https://learn.microsoft.com/en-us/training/modules/configure-azure-container-registry-container-app-deployments/4-examine-registry-operations-image-management

rg=$(az group list --query "[].name" -o tsv)
loc="eastus"

az acr create --resource-group $rg$ --name edulabsacr --sku Premium

az acr login --name edulabsacr

docker pull mcr.microsoft.com/hello-world

docker tag mcr.microsoft.com/hello-world edulabsacr.azurecr.io/hello-world:v1

docker push edulabsacr.azurecr.io/hello-world:v1

docker rmi edulabsacr.azurecr.io/hello-world:v1

docker pull edulabsacr.azurecr.io/hello-world:v1

az acr repository list -n edulabsacr -o table

az acr repository delete --name edulabsacr --image hello-world:v1


https://learn.microsoft.com/en-us/training/modules/configure-azure-container-registry-container-app-deployments/5-examine-authentication-managed-identity

az identity create -g $rg -n uami-apl2003