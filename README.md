# Pivotal Platform 2.8 on Azure

Pivotal Platform Installation on Azure

- Pivotal Application Service

## Description

## Demo

## Features

- feature:1
- feature:2

## Requirement

## Usage

## Installation
### 0. Prerequiste
### 0.1. Azure CLI
```
$ brew update
$ brew install azure-cli
```

```
$ az login
```

#### 0.1.1. Azure ID

|ID Name|Command|
|-----|-------|
|SUBSCRIPTION-ID|az account list \| jq -r '.[0].id'|
|TENANT-ID|az account list \| jq -r '.[0].tenantId'|

### 0.2. Jumpbox
#### 0.2.1. Create Resource Group
```
$ az group create --name jumpbox --location japaneast
$ az configure --defaults group=jumpbox
```

```
$ az group list
```

#### 0.2.2. Create VM for Jumpbox
```
$ az vm create \
    --resource-group jumpbox \
    --name jumpbox \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

```
$ az vm list
```

#### 0.2.3. Login Jumpbox VM
```
$ az vm list-ip-addresses|jq -r .[0].virtualMachine.network.publicIpAddresses[0].ipAddress
```

```
$ ssh azureuser@<PUBLIC_IP>
```

### 0.3. Jumpbox VM
#### 0.3.1. Azure CLI for Ubuntu
```
$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### 0.3.2. CLI for Pivotal
```
$ cd /tmp
$ sudo apt update
$ export BOSH_CLI_VERSION=6.1.1
$ export OM_VERSION=4.4.0
$ export PIVNET_CLI=0.0.76
$ wget https://github.com/cloudfoundry/bosh-cli/releases/download/v${BOSH_CLI_VERSION}/bosh-cli-${BOSH_CLI_VERSION}-linux-amd64
$ wget https://github.com/pivotal-cf/om/releases/download/${OM_VERSION}/om-linux-${OM_VERSION}
$ wget https://github.com/pivotal-cf/pivnet-cli/releases/download/v${PIVNET_CLI}/pivnet-linux-amd64-${PIVNET_CLI}
$ sudo mv bosh-cli-* /usr/local/bin/bosh
$ sudo mv om-linux-* /usr/local/bin/om
$ sudo mv pivnet-linux-* /usr/local/bin/pivnet
$ sudo chmod +x /usr/local/bin/bosh
$ sudo chmod +x /usr/local/bin/om
$ sudo chmod +x /usr/local/bin/pivnet
$ sudo apt-get -y install jq
$ sudo apt-get -y install unzip
$ sudo apt-get -y install tmux
$ sudo snap install terraform
```

#### 0.4.1. PAS Download
```
$ pivnet login --api-token='2296.........'
$ pivnet releases -p elastic-runtime
$ pivnet product-files -p elastic-runtime -r 2.8.0
```

Azure Terraform Templates
```
$ pivnet download-product-files -p elastic-runtime -r 2.8.0 -i 494978
```

Pivotal Application Service
```
$ pivnet download-product-files -p elastic-runtime -r 2.8.0 -i 560206
```

#### 0.4.2. Terraform Azure for PAS
terraform.tfvars
```
subscription_id       = "YOUR-SUBSCRIPTION-ID"
tenant_id             = "YOUR-TENANT-ID"
client_id             = "YOUR-SERVICE-PRINCIPAL-ID"
client_secret         = "YOUR-SERVICE-PRINCIPAL-PASSWORD"

env_name              = "pcf"
env_short_name        = "az"
location              = "japaneast"
ops_manager_image_uri = "https://opsmanagersoutheastasia.blob.core.windows.net/images/ops-manager-2.8.0-build.187.vhd"
dns_suffix            = "syanagihara.cf"
vm_admin_username     = "admin"
```

|Input|Command|
|-----|-------|
|SUBSCRIPTION-ID|az account list \| jq -r '.[0].id'|
|TENANT-ID|az account list \| jq -r '.[0].tenantId'|
|SERVICE-PRINCIPAL-NAME|az ad sp list --display-name boshsyanagihara \| jq -r '.[0].appId'|
|SERVICE-PRINCIPAL-PASSWORD|Swordfish|

##### AzureRM Provider ISSUE
- [azurerm provider 1.34 failds to deploy vm with image copy from blob](https://github.com/terraform-providers/terraform-provider-azurerm/issues/4361)

- Modify **main.tf**

```
provider "azurerm" {
  version = "= 1.32"
}
```

#### 0.4.3. Terraform apply
```
$ terraform init
$ terraform plan -out=plan
$ terraform apply plan
```

#### 0.4.4. DNS Record
```
$ terraform output -json | jq -r .env_dns_zone_name_servers.value
```

```
[
  "ns4-06.azure-dns.info.",
  "ns3-06.azure-dns.org.",
  "ns2-06.azure-dns.net.",
  "ns1-06.azure-dns.com."
]
```

Configure the Name Servers into DNS Provider, like Route 53, from Terraform output


## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
