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

### 0.4. Terraform for PAS
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

|Item|Value|
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

### 1.0. BOSH Director on Azure
#### 1.0.1. OpsManager Access

- https://<OPS_MANAGER_DNS>

```
$ terraform output -json | jq -r .ops_manager_dns.value
```

#### 1.0.2. Azure Config

|Item|Value|
|-----|-----|
|Subscription ID|terraform output -json \| jq -r .subscription_id.value|
|Tenant ID|terraform output -json \| jq -r .tenant_id.value|
|Application ID|terraform output -json \| jq -r .client_id.value|
|Client Secret|terraform output -json \| jq -r .client_secret.value|
|Resource Group Name|terraform output -json \| jq -r .pcf_resource_group_name.value|
|BOSH Storage Account Name|terraform output -json \| jq -r .bosh_root_storage_account.value|
|Storage Account Type|Premium_LRS|
|Default Security Group|terraform output -json \| jq -r .bosh_deployed_vms_security_group_name.value|
|SSH Public Key|terraform output -json \| jq -r .ops_manager_ssh_public_key.value|
|SSH Private Key|terraform output -json \| jq -r .ops_manager_ssh_private_key.value|
|Azure Sets|Availability Zones|
|Azure Environment|Azure Commercial Cloud|

#### 1.0.3. Director Config

|Input|Value|
|-----|-----|
|NTP Servers|0.jp.pool.ntp.org,1.jp.pool.ntp.org,2.jp.pool.ntp.org,3.jp.pool.ntp.org|
|Bosh HM Forwarder IP Address|---|
|Enable VM Resurrector Plugin|TRUE|
|Enable Post Deploy Scripts|TRUE|
|Recreate all VMs|TRUE|
|Recreate All Persistent Disks|TRUE|
|Enable bosh deploy retries|TRUE|
|Skip Director Drain Lifecycle|TRUE|
|Store BOSH Job Credentials on tmpfs (beta)|TRUE|
|Keep Unreachable Director VMs|FALSE|
|HM Pager Duty Plugin|FALSE|
|HM Email Plugin|FALSE|
|CredHub Encryption Provider|Internal|
|Blobstore Location|Internal|
|Enable TLS|TRUE|
|Database Location|Internal|
|Director Workers|5|
|Max Threads|---|
|Director Hostname|---|
|Custom SSH Banner|---|
|Identification Tags|---|

#### 1.0.4. Create Networks

|Input|Value|
|-----|-----|
|Enable ICMP checks|FALSE|

##### infrastructure

|Input|Value|
|-----|-----|
|Networks Name|infrastructure|
|infrastructure - Azure Network Name|NETWORK-NAME/SUBNET-NAME <br><br> NETWORK-NAME = terraform output -json\|jq -r .network_name.value <br> SUBNET-NAME = terraform output -json\|jq -r .management_subnet_name.value|
|infrastructure - CIDR|terraform output -json\|jq -r .management_subnet_cidrs.value[0]|
|infrastructure - Reserved IP Ranges|terraform output -json\|jq -r .management_subnet_cidrs.value[0]\|sed 's\|0/26$\|1\|g' <br> terraform output -json\|jq -r .management_subnet_cidrs.value[0]\|sed 's\|0/26$\|9\|g'|
|infrastructure - DNS|168.63.129.16|
|infrastructure - Gateway|terraform output -json\|jq -r .infrastructure_subnet_gateway.value|

##### pas

|Input|Value|
|-----|-----|
|Networks Name|pas|
|pas - Azure Network Name|NETWORK-NAME/SUBNET-NAME <br><br> NETWORK-NAME = terraform output -json\|jq -r .network_name.value <br> SUBNET-NAME = terraform output -json\|jq -r .pas_subnet_name.value|
|pas - CIDR|terraform output -json\|jq -r .pas_subnet_cidrs.value[0]|
|pas - Reserved IP Ranges|terraform output -json\|jq -r .pas_subnet_cidrs.value[0]\|sed 's\|0/22$\|1\|g' <br> terraform output -json\|jq -r .pas_subnet_cidrs.value[0]\|sed 's\|0/22$\|9\|g'|
|pas - DNS|168.63.129.16|
|pas - Gateway|terraform output -json\|jq -r .pas_subnet_gateway.value|

##### services

|Input|Value|
|-----|-----|
|Networks Name|services|
|services - Azure Network Name|NETWORK-NAME/SUBNET-NAME <br><br> NETWORK-NAME = terraform output -json\|jq -r .network_name.value <br> SUBNET-NAME = terraform output -json\|jq -r .services_subnet_name.value|
|services - CIDR|terraform output -json\|jq -r .services_subnet_cidrs.value[0]|
|services - Reserved IP Ranges|terraform output -json\|jq -r .services_subnet_cidrs.value[0]\|sed 's\|0/22$\|1\|g' <br> terraform output -json\|jq -r .services_subnet_cidrs.value[0]\|sed 's\|0/22$\|9\|g'|
|services - DNS|168.63.129.16|
|services - Gateway|terraform output -json\|jq -r .services_subnet_gateway.value|


#### 1.0.5. Assign AZs and Networks

|Input|Value|
|-----|-----|
|Singleton Availability Zone|any|
|Network|infrastructure|

#### 1.0.6. Security

|Input|Value|
|-----|-----|
|Trusted Certificates|---|
|Include OpsManager Root CA in Trusted Certs|FALSE|
|Generate VM passwords or use single password for all VMs|Generate passwords|

#### 1.0.7. BOSH DNS Config

|Input|Value|
|-----|-----|
|Excluded Recursors|---|
|Recursor Timeout|---|
|Handlers|[]|

#### 1.0.8. Syslog

|Input|Value|
|-----|-----|
|Do you want to configure Syslog for Bosh Director?|No|

#### 1.0.9. Resource Config

|Input|Value|
|-----|-----|
|BOSH Director|Standard_DS2_v2|
|Master Compilation Job|Standard_F4s|

### 1.1. PAS on Azure
#### 1.1.1. OM Authentication

```
export OPS_MGR_DNS=`terraform output -json| jq -r .ops_manager_dns.value`

export OPS_MGR_USR=
export OPS_MGR_PWD=
```

```
$ om --target https://$OPS_MGR_DNS --skip-ssl-validation configure-authentication --username $OPS_MGR_USR --password $OPS_MGR_PWD --decryption-passphrase $OPS_MGR_PWD
```

#### 1.1.2. PAS Upload

```
$ om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD --request-timeout 3600 upload-product -p cf-2.8.*.pivotal
```

#### 1.1.3. PAS Staging

```
$ om --target https://$OPS_MGR_DNS -k -u $OPS_MGR_USR -p $OPS_MGR_PWD stage-product -p cf -v 2.8.0
```

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
