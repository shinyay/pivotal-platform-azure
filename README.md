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

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
