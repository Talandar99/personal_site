+++
title = 'Introduction to terraforming clouds'
date = 2025-06-20
draft = false
+++
## Infrastructure as a code - Why Terraform tech is great
Terraform lets you manage and automate your infrastructure by writing simple, human-readable code. \
Instead of manually setting up servers, networks, or cloud services, you declare what you want in a configuration file. \
Terraform takes care of creating, updating, or deleting those resources across different platforms, making infrastructure 
management consistent, repeatable, and easy to track. The only thing that I regret about it is that i did not learned it earlier

### Terraform or OpenTofu
Terraform *WAS* open source ,but starting from version 1.6, they switched to a Business Source License (BSL), 
which restricts how the software can be used and modified, so it’s no longer fully open source.

OpenTofu is a fork of terraform created as a response to this license change. 
Tofu was created to keep the tool fully open source under a permissive license. 
It aims to maintain compatibility with Terraform configurations and providers 
while encouraging open collaboration and development without licensing restrictions.\
So if you have a choice, use tofu instead of terraform

---
## Installation and configuration
In this example we will create cheapest possible linode nanode archlinux instance 
### Installation
Use your favourite package manager to install terraform/opentofu
### Linode
1. Create linode account or use an existing one. Last time i checked they were giving 50$ for first month of usage to try their services
2. Create new ssh key. Terraform will later associate key with your username
    - Go to linode website > Upper right corner > ssh keys > (Add ssh key)
3. Create API key for using linode outside their site
    - Go to linode website > Upper right corner > API tokens > Create a presonal Access Token 
    - Give "No Access" to everything except Events, Images and Linodes. Events, Images and Linodes should be read/write (Or at least that is what i've selected)
### All needed files
1. Create new directory for project 
2. Inside this directory create `secrets.auto.tfvars` file for your secrets
```terraform
linode_token   = "token_that_looks_like_random_numbers" #<------ API TOKEN form linode 
ssh_username   = "Example99"                            #<------ here should be your linode username
root_password  = "perfectly_safe_password"              #<------ password to cloud machine created using terraform
```
3. Create another file `main.tf` with our configuration. I will explain what does what in comments
```terraform
# we don't want to accidently share our secrets, that's why we hide them in separate directory
# and mark them as "sensitive"
variable "linode_token" {
  type        = string
  sensitive   = true
}

variable "ssh_username" {
  type        = string
  sensitive   = true
}

variable "root_password" {
  type        = string
  sensitive   = true
}
# in this example we are using linode so our provider is linode
terraform {
  required_providers {
    linode = {
      source = "linode/linode"
      version = "2.5.2"
    }
  }
}

# this is where terraform need linode api token, for creating linode instance
provider "linode" {
        token = var.linode_token
}
resource "linode_instance" "web" {
  label            = "arch_eu-central_nanode-1cpu-1ram_tf"  #<------ name that will display in linode dashboard
  group            = "Terraform"

  image            = "linode/arch"                          #<------ operating system. arch means ArchLinux
  region           = "eu-central"                           #<------ server location for nanode
  type             = "g6-nanode-1"                          #<------ codename for cheapest nanode on linode 
  authorized_users = [var.ssh_username]                     #<------ here are our linode username extracted from secrets
  root_pass        = var.root_password                      #<------ here is our password extracted from secret file
  private_ip       = false

# this is definition of connection that provisioner (down bellow) uses for executing commands
  connection {
    type        = "ssh"
    user        = "root" 
    password    = var.root_password                         #<------ here is our password once again. In this case we use it to connect to resource
    host        = self.ip_address
  }
# afrer creating new machine we can send some extra commands 
# for example code bellow updates archlinux, install docker + docker compose and rebooting
  provisioner "remote-exec" {
    inline = [
      "sudo pacman -Sy archlinux-keyring --noconfirm",
      "sudo pacman -Syu git docker docker-compose neovim zip wget tmux --noconfirm",
      "sudo systemctl enable docker.service",
      "sudo reboot",
    ]
  }
}
```
### Initializing tf
1. (optional but recomended) Create alias and add it to your `~/.bashrc` file. 
```bash
alias tf=tofu
#or for terraform
alias tf=terraform
```
next commands assume you use `tf` alias

2. Initialize all resources needed for running terraform
```bash
tf init   
```
---
## Usage 
### apply configuration (run linode instance)
```bash
tf apply
```
### show current running configuration
```bash
tf show
```
### show current running configuration in json format
```bash
tf show -json
# if you have jq tool installed you can utilize it to quickly check something, like ip addres
tf show -json | jq ".values.root_module.resources[].values.ipv4"
```
### remove everything created with configuration above
```bash
tf destroy
```
---
## How do i know where to find details about os, region or plan (Linode)?
When setting up linode manually scroll down to the site, and you will see button called "View Code Snippet". 
