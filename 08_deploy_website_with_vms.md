# Deploy a website with Azure virtual machines

Vms are a scalable computing resource that give you 'total control', with no capex costs. Iaas baby!

## Decide what VM you want

The process of deciding what kind of VM you want:

- Network - what subnet/VNet will the VM be on? Will the VM get a public IP adress?
  - Consider setting up NSG (network security groups) to control traffic from/to subnets
- Name - for example: dev-euwest-01-website
- Size - A general preconfiguration of a mix of compute/memory/storage
- Pricing model - you pay per minute your VM is running, storage costs even if VM is not running
- You can save money with a reserved VM instance
- Storage - either use an unmanaged disk or a Managed disk
- Unmanaged disk means you have to configure your storage account yourself
- Managed disk means Azure will manage the storage account for you, and scale automatically
- Operating system - windows has higher hourly compute pricing than linux

## Creating a VM

The most common options are:

- Azure Portal
- Powershell/CLI
- Azure REST API - example in javascript: `azure.virtualMachines().define('my_vm_name').create()`
- Resource Manger Templates - you can export a template from an existing VM
- Azure Automation
- Azure VM Extensions - allows task on VM after deployment like installing software. Can be run in CLI/Powershell/Portal/Resource Manager templates

## Availability

Thin about the SLA of your VM. Azure reccommends an *availability set* to reduce a single point of failure. They offer 99.95% SLA on multiple-instance VM. That means you have to have at least 2 instances of the same VM in the availability set.

### Fault domain

Logical group of hardware that undergoes maintenance at the same time. Availability sets are automatically placed in different fault domains.

### Azure Site Recovery

You can also deploy accross different datacenters with Azure SiteRecovery. If one site has an outage, it will switch to the second site without interruption. You can then go back to primary site once it's back up. This should all go automatically. and can include custom powershell scripts etcetera.

You replicate the entire infra, and only run it if the primary site fails, and shut it off when the primary site is working again.

## Azure backup

![Informative image](https://docs.microsoft.com/en-us/learn/modules/intro-to-azure-virtual-machines/media/6-backup-server.png)

## Specific VM sizes

- General use/web: B, Dsv3, Dv3, DSv2, Dv2
- Heavy Computation: Fsv2, Fs, F
- Large memory usage: Esv3, Ev3, M, GS, G, DSv2, Dv2
- Data storage/ IO: Ls
- Graphics rendering: NV, NC, NCv2, NCv3, ND
- High performance computing: H

## Creation of VM

After creating an ubuntu VM, you need to do the following to initialize the disk and mount it on /data:

```bash
sudo fdisk /dev/sdc
# Press n, p, 1, enter, enter, w through the prompts
sudo mkfs -t ext4 /dev/sdc1
sudo mkdir /data & sudo mount /dev/sdc1 /data
```

Then, install apache2 or nginx or some such, and open the neccecary port (by default, only 22 is open, all ports are open within the virtual network). In the portal, go to networking, then 'add inbound port rule'. Afterwards, you should probably lock down port 22 (administrative port), or at least ip-whitelist it.

## Creation of Windows VM

Kinda similar, but with port 389 (RDP) instead. Also, you create a data disk insitead of mounting a disk (don't just use D:/, data disk is better)

## MEAN stack

MongoDB, Express, AngularJS, Node.js

Used for:

- semistructed/json-like data, not SQL
- well documented, runs on windoes/linux/mac
- You are confortable in javascript

Instructions:

- First, create a resource group with `az group create`
- Then, create a VM with `az vm create --name vmname`
  - (these need other params like resource group, image, admin username, etcetera)
- Then, open port 80 with `az vm open-port --port 80 --name vmname`
- Then, get the IP address with `az vm show --name vmname --show-details --query [publicIps]`
- Then ssh to it with the ssh keys you generated, and `sudo apt install` mongodb, npm, etcetera
- `exit`, then move to your local webapp with angular and express, here at `~/Books`
- copy the files: `scp -r ~/Books user@$ipaddress:~/Books`
- Then, ssh to the server, `cd ~/Books`, and `npm install`
- Then, run the server with `sudo node server.js`
- Then, visit it in the browser by visiting the ip adress
