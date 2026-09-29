# Ansible Role: gcp_network

Creates Google Cloud VPC networks, regional subnets, and an IAP-only SSH
firewall rule.

### Features

- Creates multiple custom VPC networks and one or more regional subnets per
  network.
- Creates an SSH firewall rule restricted to the Google Cloud IAP TCP
  forwarding range.
- Can be run before VM provisioning to prepare shared network infrastructure.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role has no Ansible role dependencies. The Compute Engine API must be enabled in the target project. |

### Optional Roles

| Role | Required | Purpose |
| --- | --- | --- |
| `gcp_project` | No | Creates the project and enables the Compute Engine API before network provisioning. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- An existing Google Cloud project with the Compute Engine API enabled, or the
  `gcp_project` role earlier in the play
- Permissions to create Compute Engine networks, subnetworks, and firewall
  rules

Install the collection from the repository root with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Supported Platforms

| Platform | Version |
| --- | --- |
| Ansible controller | Any platform supported by Ansible |

## Supported Clouds

| Cloud Platform | Supported |
| --- | --- |
| GCP | Yes |

## Usage

Include the role in a playbook that connects locally to Google Cloud. Run
`gcp_project` first when the project or Compute Engine API needs to be created:

```yaml
- name: Provision Google Cloud networks
  hosts: gcp_provision
  gather_facts: false
roles:
  - role: gcp_project
  - role: gcp_network
```

The repository entry point is `playbooks/create_gcp_vm.yml`, which runs
`gcp_network` before `gcp_vm`. Provisioning configuration is in
`inventory/group_vars/gcp_provision.yml`.

## Configuration

The recommended configuration is `inventory/group_vars/gcp_provision.yml`. Set
`gcp_network_project_id` there, or use the existing `gcp_vm_project_id` setting
from the repository inventory. Define `gcp_network_networks` with one or more
networks; each network needs a `name` and a non-empty `subnets` list. Every
subnet needs a `name`, `region`, and `ip_cidr_range`:

```yaml
gcp_network_project_id: "your-project-id"
gcp_network_networks:
  - name: app-network
    subnets:
      - name: app-europe
        region: europe-west3
        ip_cidr_range: 10.10.0.0/24
      - name: app-us
        region: us-central1
        ip_cidr_range: 10.20.0.0/24
  - name: data-network
    subnets:
      - name: data-europe
        region: europe-west3
        ip_cidr_range: 10.30.0.0/24
gcp_network_ssh_firewall_network: app-network
```

The defaults create one network and subnet (`gcp-ansible-network`,
`gcp-ansible-subnet`, `10.10.0.0/24`) in `europe-west3`. Network, subnet, and
region defaults can be overridden with `GCP_NETWORK`, `GCP_SUBNET`, and
`GCP_REGION`. The project ID defaults to `gcp_vm_project_id` for compatibility
with the existing inventory, or can be set with `gcp_network_project_id` or
`GCP_PROJECT_ID`.

The role creates one IAP-only SSH firewall rule on the network named by
`gcp_network_ssh_firewall_network`. By default, this is the default network.
When a VM uses another network, set this variable to the same network name. The
firewall name can be overridden with `GCP_FIREWALL`; source ranges and target
tags can be set with `gcp_network_ssh_firewall_source_ranges` and
`gcp_network_ssh_firewall_target_tags`.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_network_project_id` | `gcp_vm_project_id`, otherwise `GCP_PROJECT_ID` | Google Cloud project in which resources are created. |
| `gcp_network_auth_kind` | `application` | Credential type used by the Google Cloud modules. |
| `gcp_network_default_region` | `europe-west3` | Region used by the default subnet. Can be overridden with `GCP_REGION`. |
| `gcp_network_default_name` | `gcp-ansible-network` | Name used by the default VPC. Can be overridden with `GCP_NETWORK`. |
| `gcp_network_default_subnet_name` | `gcp-ansible-subnet` | Name used by the default subnet. Can be overridden with `GCP_SUBNET`. |
| `gcp_network_networks` | One VPC with subnet `10.10.0.0/24` | VPCs and their subnet definitions. Each subnet specifies a name, region, and CIDR range. |
| `gcp_network_ssh_firewall_name` | `gcp-ansible-allow-ssh` | Name of the IAP SSH firewall rule. Can be overridden with `GCP_FIREWALL`. |
| `gcp_network_ssh_firewall_network` | `gcp_network_default_name` | VPC to which the SSH firewall rule applies. |
| `gcp_network_ssh_firewall_source_ranges` | `35.235.240.0/20` | Source ranges allowed by the SSH firewall rule. 35.235.240.0/20 is the IPv4 source range for IAP TCP Forwarding |
| `gcp_network_ssh_firewall_target_tags` | `ssh` | Network tags targeted by the SSH firewall rule. |

Role variables use the `gcp_network_` prefix and can also be overridden
directly in a playbook, for example:

```yaml
roles:
  - role: gcp_network
    vars:
      gcp_network_ssh_firewall_source_ranges:
        - 35.235.240.0/20
```

The `gcp_network` role can be used without `gcp_vm`; in that case, set
`gcp_network_project_id` directly or provide `GCP_PROJECT_ID`.

## Encrypted Data / Ansible Vault

The role does not require encrypted variables or Ansible Vault. You can encrypt
project-specific values or credentials in your inventory using the standard
Ansible Vault workflow if required by your environment.

## Tags

Available tags:

| Tag | Purpose |
|-------|---------|
| None | No role-specific task tags are defined. |

## Known Limitations

- The role creates an IAP SSH firewall rule on one configured network; it does
  not create separate firewall rules for every VPC.
- The SSH rule allows TCP port 22 only from the IAP TCP forwarding range by
  default. Direct SSH from the internet requires a separate firewall rule.
- The role does not configure VPC peering, routes, Cloud NAT, or other network
  services.
- The role does not delete VPCs, subnets, or firewall rules.

## Run Ansible

After configuring `inventory/group_vars/gcp_provision.yml`, run the repository
playbook:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_vm.yml
```

To validate the playbook without contacting Google Cloud, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini --syntax-check playbooks/create_gcp_vm.yml
```

## Deletion

Delete dependent resources before their VPC network. For example:

```bash
gcloud compute networks subnets delete app-europe \
  --region europe-west3 \
  --project your-project-id
gcloud compute firewall-rules delete gcp-ansible-allow-ssh \
  --project your-project-id
gcloud compute networks delete app-network \
  --project your-project-id
```

Check for other resources attached to a network before deleting it.