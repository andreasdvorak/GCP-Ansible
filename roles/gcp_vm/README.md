# gcp_vm

Creates a Google Cloud VPC network, subnet, SSH firewall rule, and Compute Engine VM.

## Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- A Google Cloud project with the Compute Engine API enabled
- Permissions to create Compute Engine networks, firewall rules, and instances
- The Google Cloud CLI (`gcloud`) installed and authenticated on the controller

Install the collection from the repository root with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Usage

Include the role from a playbook that runs on `localhost` with a local connection:

```yaml
- name: Provision a Google Cloud VM
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: gcp_vm
```

The repository entry point is `playbooks/create_gcp_vm.yml`. The role is configured for the `localhost` host through `inventory/group_vars/localhost.yml`.

## Configuration

The recommended configuration is `inventory/group_vars/localhost.yml`. Set the project-specific `gcp_vm_project_id` there. The role defaults cover the remaining settings and only need to be overridden when required:

```yaml
gcp_vm_project_id: "your-project-id"
```

The role also supports environment variables as fallbacks for the project and VM settings. `GCP_PROJECT_ID` is required when no `gcp_vm_project_id` is set. `gcp_vm_auth_kind` selects the credential type used by the Google Cloud modules and should remain `application` when using Application-Default-Credentials.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_vm_os` | `linux` | Target operating system; `linux` is implemented and `windows` is prepared but not implemented. |
| `gcp_vm_function` | inventory host name | Function identifier used in the `function` label and `function-<value>` network tag. |
| `gcp_vm_labels` | `managed-by: ansible` | Labels applied to the VM. |
| `gcp_vm_network_tags` | `ssh`, `ansible-managed`, `function-<value>` | Network tags applied to the VM. |
| `gcp_vm_project_id` | required | Google Cloud project in which the resources are created. |
| `gcp_vm_auth_kind` | `application` | Credential type; `application` uses Application Default Credentials. |
| `gcp_vm_region` | `europe-west3` | Region for the subnet. |
| `gcp_vm_zone` | `europe-west3-a` | Zone for the Compute Engine VM. |
| `gcp_vm_network_name` | `gcp-ansible-network` | Name of the custom VPC network. |
| `gcp_vm_subnet_name` | `gcp-ansible-subnet` | Name of the subnet in the VPC network. |
| `gcp_vm_firewall_name` | `gcp-ansible-allow-ssh` | Name of the SSH firewall rule. |
| `gcp_vm_instance_name` | `gcp-ansible-vm` | Name of the Compute Engine VM. |
| `gcp_vm_machine_type` | `e2-micro` | Machine type and VM size. |
| `gcp_vm_image_project` | `debian-cloud` | Google Cloud project providing the VM image family. |
| `gcp_vm_image_family` | `debian-12` | Operating system image family to use. |
| `gcp_vm_ssh_user` | `ansible` | Linux user created by the Compute Engine guest agent. |
| `gcp_vm_ssh_public_key_file` | `.ssh/gcp_linux.pub` | Controller-side public SSH key added for the user. |

Role variables use the `gcp_vm_` prefix and can also be overridden directly in a playbook, for example:

```yaml
roles:
  - role: gcp_vm
    vars:
      gcp_vm_machine_type: e2-small
      gcp_vm_instance_name: development-vm
```

The operating system can be selected with `gcp_vm_os: linux` or `gcp_vm_os: windows`. Linux provisioning is implemented. Windows currently has a separate task entry point and fails with an explicit not-implemented message until Windows image, network, and management settings are defined.

The role does not create an external IPv4 address. SSH is permitted from the Google IAP TCP forwarding range (`35.235.240.0/20`); grant the required IAP tunnel and OS Login permissions before connecting.

The public key in `gcp_vm_ssh_public_key_file` is added to the instance metadata as `ssh-keys`. The Compute Engine guest agent creates the `ansible` user and its `authorized_keys` entry on the VM. The corresponding private key must remain on the controller and is not managed by this role.

GCP distinguishes between labels and network tags. The VM receives the labels `managed-by=ansible` and `function=mytest`, plus the network tags `ansible-managed` and `function-mytest` for the current `mytest` host. VPC networks, subnets, and firewall rules do not support the same VM network tags; their names and the managed VM label are used for identification instead.

## Provisioning

After configuring `inventory/group_vars/localhost.yml`, run:

```bash
ansible-playbook playbooks/create_gcp_vm.yml
```

For an initial dry run:

```bash
ansible-playbook playbooks/create_gcp_vm.yml --check
```

`--check` does not fully replace an API permission check. After provisioning, inspect the VM with:

```bash
gcloud compute instances describe gcp-ansible-vm \
  --zone europe-west3-a \
  --project your-project-id
```

## Deletion

VM deletion is deliberately separate from provisioning:

```bash
gcloud compute instances delete gcp-ansible-vm \
  --zone europe-west3-a \
  --project your-project-id
```

The network and firewall rule remain in place and can be removed separately through the Google Cloud console or `gcloud`.