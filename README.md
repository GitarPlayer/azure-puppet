---
page_type: sample
languages:
- json
products:
- azure
- terraform
- puppet
description: "A quick setup in Azure to learn puppet done with terraform."
---

# Learn puppet on azure
 
This repo contains the code to quickly spin up two CentOS VMs on Azure. One of them can be configured as the puppet server and on the other we install the puppet agent. **Warning: This code is not meant for use in a production environment (password authentication for the VMs).**

The following files are included:

|File name|Description|
|---|---|
|*main.tf*|The teraform code that setups the VMs.|
|*outputs.tf*|The terraform file that makes sure that the public ips of the VMs will be printed after the terraform apply command. |



## Prerequisites

- a valid azure subscription
- az cli 2.24.2
- Terraform v1.0.0

