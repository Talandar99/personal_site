+++
title = 'Creating Proxmox VM with Terraform'
date = 2025-08-07
draft = false
+++
## Prerequisites
I assume you have Proxmox already installed and you know how to create VM in cloud using terraform

---
## Creating VM template
You need VM template for creating vm's with terraform
### Get standard installation ISO image 
- for this example we will use debian netinst iso.
[link to debian page](https://www.debian.org/distrib/)
- you can dowload iso directly to proxmox 
proxmox web ui -> node_name -> Local -> "Download from url"
---
### Create standard VM
go to proxmox web interface
1. Create VM button 

2. General
 - Node: your node name
 - Name: does not matter
 - VM ID: does not matter

3. OS 
 - Storage: local
 - ISO: iso with installation of your system
 - Type: Linux
 - Kernel: leave default

4. System
 - Qemu Agent = true
 - Leave rest

5. Disks
 - does not matter but give at least 10GB

6. CPU 
 - does not matter 

7. Memory 
 - does not matter

8. Web
 - Bridge: vmbr0
 - Model: VirtiO

9. Confirm
 - Start After Created

---
### System Installation
- Install System. You don't need gui, but ssh server will be usefull.
- When picking password pick something simple , we will change it later when provisionning with terraform
#### Install cloud init package
Cloud-init enables automated setup of things like users, SSH keys, packages, and scripts when a VM starts for the first time.
In our case cloud init will be used by Terraform.
```bash
    apt update
    apt install cloud-init qemu-guest-agent -y
    systemctl enable qemu-guest-agent
```
 - remember about installing ssh server
 - other stuff does not matter too much, just make sure everything you wanted is installed
#### Execute this if you want to log in as a root via ssh 
(technicly you can do everything without it but it's easier this way)
```bash
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config 
echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config 
```
#### Clear image 
(optional but recomended)
```bash
   cloud-init clean
   truncate -s 0 /etc/machine-id
```
---
### Prepare template
1. shutdown vm
2. Hardware -> Add -> CloudInit Drive
 - Storage: local-lvm
3. Options
 - make sure scsi0 is first in boot order
 - Qemu Guest Agent = Enabled
#### Turn into template
More -> Convert to template

---
## Using terraform to setup VM or multiple VM's
for this example we will create 2 VM's with static ip address. 
Copy configurations bellow and update things specified by comments
### secrets file
create secrets.auto.tfvars file
```terraform
pm_address                  = ""    # your proxmox ip address
ssh_password                = ""    # new ssh password for vms
pm_password                 = ""    # password for your proxmox
```
### main configuration
create main.tf file
```terraform
# variables
variable "instances" {
  description = "list of vms to create"
  type = map(object({
    ip   = string
    vmid = number
  }))
  default = {
    "vm1" = {
      ip   = "192.168.1.81/24" # ip address will be static but it have to be unique
      vmid = 181               # vmid should also be unique to avoid nuke'ing another VM in proxmox by accident
    },
    "vm2" = {
      ip   = "192.168.1.82/24" # ip address will be static but it have to be unique
      vmid = 182               # vmid should also be unique to avoid nuke'ing another VM in proxmox by accident
    }
  }
}
variable "ssh_username" {
  type    = string
  default = "root"
}
variable "ssh_password" {
  type      = string
  sensitive = true
}
variable "pm_password" {
  type      = string
  sensitive = true
}

variable "pm_address" {
  type      = string
  sensitive = true
}

# providers
terraform {
  required_providers {
    proxmox = {
      source  = "Telmate/proxmox"
      version = "3.0.2-rc01"
    }
  }
}

provider "proxmox" {
  pm_api_url      = "https://${var.pm_address}:8006/api2/json" 
  pm_user         = "root@pam"
  pm_password     = var.pm_password
  pm_tls_insecure = true
}

# resource
resource "proxmox_vm_qemu" "vms" {
  for_each = var.instances

  name        = each.key
  target_node = "proxmox"               # this is name of proxmox node, yours can be different
  vmid        = each.value.vmid
  clone       = "debian-12-template"    # this is your template name, yours can be different

  cpu {
    cores = 2                           # here you can specify ammount of cores used by VM
  }

  memory    = 2048                      # here you can specify RAM
  agent     = 1
  skip_ipv6 = true

  os_type    = "cloud-init"
  ciuser     = var.ssh_username
  cipassword = var.ssh_password

  ipconfig0 = "ip=${each.value.ip},gw=192.168.1.1"

  disk {
    type    = "disk"
    size    = "10G"                     # here you can specify disk size
    storage = "local-lvm"
    slot    = "scsi0"
  }
  disk {
    type    = "cloudinit"
    storage = "local-lvm"
    slot    = "ide2"
  }
  network {
    id     = 0
    model  = "virtio"
    bridge = "vmbr0"
  }
  # provisionner can be used to run additional commands when setting up vm for first time
  provisioner "remote-exec" {
    inline = [
      "apt update -y",
      "apt upgrade -y",
      "apt install iptables git tar zip unzip curl -y",
    ]

    connection {
      type     = "ssh"
      user     = var.ssh_username
      password = var.ssh_password
      host     = split("/", each.value.ip)[0]
    }
  }
}
```
### Using OpenTofu/terraform to run
i recomend adding alias to .bashrc
```bash
alias tf=tofu
#or for terraform
alias tf=terraform
```
basic commands
```bash
# init terraform in new place
tf init    
# apply configuration
tf apply
# destroy configuration
tf destroy
```
