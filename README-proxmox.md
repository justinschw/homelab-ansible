# Proxmox Ansible Configuration

This repository includes Ansible playbooks and roles for configuring a Proxmox server for use with Packer and Terraform.

## Overview

The `provisioner` role sets up a dedicated Proxmox user account with appropriate permissions for infrastructure automation tools like Packer and Terraform. This user can:

- Create, delete, and clone Virtual Machines (VMs)
- Start and stop VMs
- Configure cloud-init enabled VMs
- Manage LXC containers
- Access the Proxmox API

## Prerequisites

- A Proxmox VE server (version 7.0 or later recommended)
- Ansible installed on your control machine
- SSH access to the Proxmox server with root or sudo privileges

## Usage

### Running the Provisioner Playbook

To set up the provisioner user on your Proxmox server:

```bash
ansible-playbook -i inventory proxmox.yml
```

### What the Playbook Does

1. Creates a Proxmox user named `provisioner`
2. Grants the user appropriate permissions for VM and LXC container management
3. Creates an API token for the provisioner user
4. Saves the API token securely to a local file in `~/.proxmox/`

### API Token Storage

The API token will be saved to:
```
~/.proxmox/provisioner_token
```

This file contains the token ID and secret needed for Packer and Terraform to authenticate with Proxmox.

## Security Notes

- The API token file is created with restricted permissions (0600)
- Store the token securely and never commit it to version control
- Consider using Ansible Vault for sensitive variables
- Rotate API tokens regularly

## Configuring Packer and Terraform

After running this playbook, you can use the generated API token in your Packer templates and Terraform configurations:

### Packer Example
```hcl
proxmox_url = "https://your-proxmox-server:8006/api2/json"
proxmox_token_id = "provisioner@pve!automation"
proxmox_token_secret = "<token-from-file>"
```

### Terraform Example
```hcl
provider "proxmox" {
  pm_api_url = "https://your-proxmox-server:8006/api2/json"
  pm_api_token_id = "provisioner@pve!automation"
  pm_api_token_secret = "<token-from-file>"
}
```

## Role Variables

See `roles/provisioner/defaults/main.yml` for configurable variables.

## License

MIT
