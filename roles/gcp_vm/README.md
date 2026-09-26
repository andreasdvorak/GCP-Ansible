# Ansible Role: gcp_vm

Creates a Google Cloud VPC network, subnet, SSH firewall rule, and Compute Engine VM.

### Features

- Creates a custom VPC network and a subnet in the configured region.
- Allows SSH through Google Cloud IAP TCP forwarding and applies the configured
  network tags to the VM.
- Creates a Linux Compute Engine VM from the configured image family and adds
  the controller's public SSH key to instance metadata.
- Can optionally assign an ephemeral external IPv4 address to the VM.
- Supports separate label management through the `gcp_vm_labels` role.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role has no Ansible role dependencies. |

### Optional Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | No | Guest OS configuration is not performed by this role. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- A configured `gcp_vm_project_id`; the repository playbook creates the project
  and enables the Compute Engine API through the `gcp_project` role
- A billing account linked to the project before creating billable resources
- Permissions to create projects and enable APIs when the project does not yet
  exist, and to create Compute Engine networks, firewall rules, and instances
- The Google Cloud CLI (`gcloud`) installed and authenticated on the controller

Install the collection from the repository root with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Supported Platforms

| Platform | Version |
| --- | --- |
| Linux (Debian image family) | Debian 12 by default |
| Windows | Not implemented |

## Supported Clouds

| Cloud Platform | Supported |
| --- | --- |
| GCP | Yes |

## Usage

Include the role from a playbook that uses the `provision` inventory group. The
inventory host connects locally while retaining its host variables:

```yaml
- name: Provision a Google Cloud VM
  hosts: provision
  gather_facts: false
  roles:
    - role: gcp_vm
```

The repository entry point is `playbooks/create_gcp_vm.yml`. Provisioning
configuration is in `inventory/group_vars/provision.yml`; host-specific values
are loaded from `inventory/host_vars/`.

## Configuration

The recommended configuration is `inventory/group_vars/provision.yml`. Set the
project-specific `gcp_vm_project_id` there. The role defaults cover the
remaining settings and only need to be overridden when required:

```yaml
gcp_vm_project_id: "your-project-id"
```

The role also supports environment variables as fallbacks for the project and VM settings. `GCP_PROJECT_ID` is required when no `gcp_vm_project_id` is set. `gcp_vm_auth_kind` selects the credential type used by the Google Cloud modules and should remain `application` when using Application-Default-Credentials.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_vm_os` | `linux` | Target operating system. Linux is implemented; Windows currently fails with an explicit not-implemented message. |
| `gcp_vm_function` | Inventory host name | Function identifier used in the `function-<value>` network tag and by the labels role. |
| `gcp_vm_network_tags` | `ssh`, `ansible-managed`, `function-<value>` | Network tags applied to the VM. |
| `gcp_vm_project_id` | `GCP_PROJECT_ID` environment variable; required if unset | Google Cloud project in which the resources are created. |
| `gcp_vm_auth_kind` | `application` | Credential type used by the Google Cloud modules. |
| `gcp_vm_region` | `europe-west3` | Region for the subnet. Can be overridden with `GCP_REGION`. |
| `gcp_vm_zone` | `europe-west3-a` | Zone for the Compute Engine VM. Can be overridden with `GCP_ZONE`. |
| `gcp_vm_network_name` | `gcp-ansible-network` | Name of the custom VPC network. Can be overridden with `GCP_NETWORK`. |
| `gcp_vm_subnet_name` | `gcp-ansible-subnet` | Name of the subnet. Can be overridden with `GCP_SUBNET`. |
| `gcp_vm_firewall_name` | `gcp-ansible-allow-ssh` | Name of the SSH firewall rule. Can be overridden with `GCP_FIREWALL`. |
| `gcp_vm_instance_name` | Inventory host name | Name of the Compute Engine VM; for example, host `mytest` uses `host_vars/mytest.yml` and creates a VM named `mytest`. |
| `gcp_vm_assign_public_ip` | `false` | Assign an ephemeral external IPv4 address when `true`. |
| `gcp_vm_machine_type` | `e2-micro` | Machine type and VM size. Can be overridden with `GCP_MACHINE_TYPE`. |
| `gcp_vm_image_project` | `debian-cloud` | Project providing the VM image family. Can be overridden with `GCP_IMAGE_PROJECT`. |
| `gcp_vm_image_family` | `debian-12` | Operating system image family. Can be overridden with `GCP_IMAGE_FAMILY`. |
| `gcp_vm_ssh_user` | `ansible` | Linux user created by the Compute Engine guest agent. |
| `gcp_vm_ssh_public_key_file` | `{{ playbook_dir }}/../.ssh/gcp_linux.pub` | Controller-side public SSH key added for the user. |

Role variables use the `gcp_vm_` prefix and can also be overridden directly in a playbook, for example:

```yaml
roles:
  - role: gcp_vm
    vars:
      gcp_vm_machine_type: e2-small
```

Ansible loads `host_vars/<inventory_hostname>.yml` for each inventory host. The
role uses that inventory hostname as the VM name by default. An explicit
`gcp_vm_instance_name` variable can still override this default.

The operating system can be selected with `gcp_vm_os: linux` or `gcp_vm_os: windows`. Linux provisioning is implemented. Windows currently has a separate task entry point and fails with an explicit not-implemented message until Windows image, network, and management settings are defined.

By default, the role does not create an external IPv4 address. Set
`gcp_vm_assign_public_ip: true` in the host or play variables to request an
ephemeral external IPv4 address. Google charges for external IPv4 addresses and
outbound internet traffic. The SSH firewall remains restricted to the Google
IAP TCP forwarding range (`35.235.240.0/20`), so assigning an IP alone does not
enable direct SSH or ping from the internet. Grant the required IAP tunnel and
OS Login permissions when connecting through IAP.

The public key in `gcp_vm_ssh_public_key_file` is added to the instance metadata as `ssh-keys`. The Compute Engine guest agent creates the `ansible` user and its `authorized_keys` entry on the VM. The corresponding private key must remain on the controller and is not managed by this role.

GCP distinguishes between labels and network tags. Labels are managed by the
separate `gcp_vm_labels` role; network tags such as `ssh` are used for firewall
targeting. See [roles/gcp_vm_labels/README.md](../gcp_vm_labels/README.md) for
label configuration and independent updates.

## Encrypted Data / Ansible Vault

The role does not require encrypted variables or Ansible Vault. You can encrypt
project-specific values or credentials in your inventory using the standard
Ansible Vault workflow if required by your environment.

## Tags

Available tags:

| Tag | Purpose |
|-------|---------|
| None | No role-specific Ansible task tags are defined. |

## Known Limitations

- Only Linux VM provisioning is implemented; the default image family is Debian
  12.
- The subnet CIDR is currently fixed at `10.10.0.0/24`.
- The VM has no external IPv4 address unless `gcp_vm_assign_public_ip` is
  enabled. The SSH firewall only allows IAP TCP forwarding from
  `35.235.240.0/20`; direct SSH and ICMP require separate firewall rules.
- The role creates a 10 GB `pd-balanced` boot disk; disk size and type are not
  exposed as role variables.
- Deleting the VM does not delete the VPC network, subnet, or firewall rule.

## Run Ansible

After configuring `inventory/group_vars/provision.yml`, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_vm.yml [--check}
```

`--check` does not fully replace an API permission check. After provisioning, inspect the VM with:

```bash
gcloud compute instances describe mytest \
  --zone europe-west3-a \
  --project your-project-id
```

## Deletion

VM deletion is deliberately separate from provisioning:

```bash
gcloud compute instances delete mytest \
  --zone europe-west3-a \
  --project your-project-id
```

The network and firewall rule remain in place and can be removed separately through the Google Cloud console or `gcloud`.