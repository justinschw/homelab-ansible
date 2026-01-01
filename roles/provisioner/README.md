# Provisioner Role

This Ansible role sets up a dedicated user account in Proxmox VE for infrastructure automation tools like Packer and Terraform.

## Description

The role performs the following tasks:

1. Creates a Proxmox user named `provisioner` (configurable)
2. Creates a custom role `PVEVMAdmin` with appropriate permissions
3. Assigns the role to the provisioner user
4. Generates an API token for the user
5. Saves the API token securely to the local filesystem

## Requirements

- Proxmox VE 7.0 or later
- Root or privileged access to the Proxmox server
- Ansible 2.9 or later

## Role Variables

All variables have sensible defaults defined in `defaults/main.yml`:

- `provisioner_user`: Username for the provisioner account (default: `provisioner`)
- `provisioner_realm`: Proxmox realm (default: `pve`)
- `provisioner_token_name`: Name for the API token (default: `automation`)
- `token_storage_dir`: Local directory for token storage (default: `~/.proxmox`)
- `token_storage_file`: Full path to token file (default: `~/.proxmox/provisioner_token`)
- `provisioner_comment`: User description (default: `Automation user for Packer and Terraform`)
- `provisioner_password`: Optional password for the user (default: empty)
- `provisioner_privileges`: List of Proxmox privileges granted to the user

## Permissions Granted

The role grants the following Proxmox privileges to the provisioner user:

- VM.Allocate - Allocate new VMs
- VM.Clone - Clone existing VMs
- VM.Config.* - Configure VM settings (CPU, memory, disk, network, cloud-init, etc.)
- VM.Monitor - Monitor VM status
- VM.Audit - View VM information
- VM.PowerMgmt - Start, stop, and restart VMs
- Datastore.AllocateSpace - Allocate storage space
- Datastore.Audit - View datastore information
- Sys.Audit - View system information
- Pool.Allocate - Create and manage resource pools

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure Proxmox provisioner user
  hosts: proxmox
  become: yes
  roles:
    - provisioner
```

## Example with Custom Variables

```yaml
---
- name: Configure Proxmox provisioner user
  hosts: proxmox
  become: yes
  roles:
    - role: provisioner
      vars:
        provisioner_user: "terraform"
        provisioner_token_name: "tftoken"
        token_storage_dir: "/var/local/tokens"
```

## Token Storage

The API token is saved to a local file with the following format:

```bash
# Proxmox API Token for provisioner@pve
# Token ID: provisioner@pve!automation
# Created: 2024-01-01T00:00:00Z

PROXMOX_TOKEN_ID="provisioner@pve!automation"
PROXMOX_TOKEN_SECRET="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Use these values in your Packer and Terraform configurations
```

The file is created with `0600` permissions to ensure only the owner can read it.

## Security Considerations

1. The API token has full privileges for VM and container management
2. Store the token file securely and never commit it to version control
3. Consider using Ansible Vault for sensitive variables
4. Rotate API tokens regularly
5. Use `--privsep 0` to ensure the token has the same privileges as the user

## License

MIT

## Author Information

Created for homelab automation with Proxmox VE, Packer, and Terraform.
