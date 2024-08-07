# Azure Tips


## **Azure CLI**

The Azure command-line interface (Azure CLI) is a set of commands used to create and manage Azure resources.


### ***Installing Azure CLI***

There are two options as mentioned at [Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt),
standalone or apt package management. For security reasons, I'll follow apt.

```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```

Installing signing key:

```
sudo mkdir -p /etc/apt/keyrings
curl -sLS https://packages.microsoft.com/keys/microsoft.asc |
  gpg --dearmor | sudo tee /etc/apt/keyrings/microsoft.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/microsoft.gpg
```

Add repo into /etc/apt/source.list.d/azure-cli.sources

```
AZ_DIST=$(lsb_release -cs)
echo "Types: deb
URIs: https://packages.microsoft.com/repos/azure-cli/
Suites: ${AZ_DIST}
Components: main
Architectures: $(dpkg --print-architecture)
Signed-by: /etc/apt/keyrings/microsoft.gpg" | sudo tee /etc/apt/sources.list.d/azure-cli.sources
```

Installing

```
sudo apt-get update
sudo apt-get install azure-cli
```

### ***Azure CLI Commands***

Searching for command related to vm

```
az find vm
```

Needing more help?

```
az vm --help
```

Hum are you not so confident? Don't worry, there is interactive mode. 

```
az interactive
```

Are you hungry? Get the latest languages and supported versions. 

```
az webapp list-runtimes --os linux
```

Couldn't find what you need? Let me see ... you can run a container!


### ***Azure Container CLI***

> [!NOTE]
> ACR Tasks is a suite of features within Azure Container Registry that provides streamlined and efficient Docker container image builds in Azure. In this article, you learn how to use the quick task feature of ACR Tasks.

Create a resource group

```
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus
```

Build container image from a sample code. ACR tasks use docker build to build your images, no changes to your Dockerfiles are required to start using ACR Tasks immediately.

```
az acr build --registry $ACR_NAME --image helloacrtasks:v1 --file /path/to/Dockerfile /path/to/build/context.
```

Create a Key Vault

```
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

Create a service principal and store its credentials in your key vault.

```
az keyvault secret set \
 --vault-name $AKV_NAME \
 --name $ACR_NAME-pull-pwd \
 --value $(az ad sp create-for-rbac \
 --name $ACR_NAME-pull \
 --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
 --role acrpull \
 --query password \
 --output tsv)
```

Store the service principal's appi in the vault

```
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --value $(az ad sp list --display-name $ACR_NAME-pull --query [].appId --output tsv)
```

Two secrets have been stored in Azure Key Vault:

- $ACR_NAME-pull-usr: The service principal ID, for use as the container registry username.
- $ACR_NAME-pull-pwd: The service principal password, for use as the container registry password.

Finally create your container!

```
az container create \
 --resource-group $RES_GROUP \
 --name acr-tasks \
 --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
 --registry-login-server $ACR_NAME.azurecr.io \
 --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) 
 --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) 
 --dns-name-label acr-tasks-$ACR_NAME 
 --query "{FQDN:ipAddress.fqdn}" 
 --output table
```

Deployment status

```
az container attach --resource-group $RES_GROUP --name acr-tasks
```

Not satisfied? Just delete it! :)

```
az container delete --resource-group $RES_GROUP --name acr-tasks
```

To remove all resources, just do the following:

```
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## **More details**
- [Learn Microsoft Azure](https://learn.microsoft.com/en-us/cli/azure/)
- [Creating containers](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)

