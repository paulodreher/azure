# Azure Tips


## **Azure CLI**

The Azure command-line interface (Azure CLI) is a set of commands used to create and manage Azure resources.


### ***Installing Azure CLI***

There are two options as mentioned at [Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt),
standalone or apt package management. For security reasons, I'll follow apt.

```
sudo apt-get update`
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

## Azure CLI Commands:

Searching for command related to vm

```
az find vm
```

Needing more help?

```
az vm --help
```

Hum are not so confident? Don't worry, there is interactive mode. 

```
az interactive
```





More details at [Learn Microsoft Azure](https://learn.microsoft.com/en-us/cli/azure/)

