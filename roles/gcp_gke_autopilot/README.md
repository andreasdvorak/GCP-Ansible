# Ansible Role: gcp_gke_autopilot

Creates a regional Google Kubernetes Engine (GKE) Autopilot cluster and
enables the Kubernetes Engine API.

### Features

- Creates multiple regional GKE Autopilot clusters from one inventory list.
- Creates nodes without public IP addresses by default, for projects where
	external VM IPs are restricted by organization policy.
- Supports shared defaults and per-cluster overrides for region, release
	channel, private nodes, VPC network, and subnet.
- Leaves existing healthy Autopilot clusters unchanged and prevents collisions
	with Standard clusters that have the same name and region.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role enables the Kubernetes Engine API itself. |

### Optional Roles

| Role | Required | Purpose |
| --- | --- | --- |
| `gcp_project` | No | Creates the project if needed and enables the Compute Engine API before cluster provisioning. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- Google Cloud CLI (`gcloud`) installed and available in `PATH`
- An existing Google Cloud project with billing enabled, or the `gcp_project`
	role earlier in the play
- Permissions to enable the Kubernetes Engine API, list clusters, and create
	GKE clusters

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
`gcp_project` first when the project needs to be created or the Compute Engine
API needs to be enabled:

```yaml
- name: Provision a Google Kubernetes Engine Autopilot cluster
  hosts: gcp_provision
  gather_facts: false
  roles:
    - role: gcp_project
    - role: gcp_gke_autopilot
```

The repository entry point is `playbooks/create_gke_autopilot.yml`. GKE
Autopilot settings are in `inventory/gke_autopilot.yml`; the project ID is
referenced from `inventory/group_vars/gcp_provision.yml`.

## Configuration

The repository playbook loads the role configuration from
`inventory/gke_autopilot.yml`. The project ID there references
`gcp_vm_project_id` from `inventory/group_vars/gcp_provision.yml`. Shared
settings are defaults for each cluster. Add one entry per cluster to
`gcp_gke_autopilot_clusters`; individual entries can override those defaults:

```yaml
gcp_gke_autopilot_project_id: your-project-id
gcp_gke_autopilot_region: europe-west3
gcp_gke_autopilot_release_channel: regular
gcp_gke_autopilot_enable_private_nodes: true
gcp_gke_autopilot_clusters:
  - name: app-autopilot
  - name: batch-autopilot
    region: europe-west1
    release_channel: stable
    network: app-network
    subnetwork: app-europe
```

Each cluster requires a name. Per-cluster `region`, `release_channel`,
`enable_private_nodes`, `network`, and `subnetwork` are optional and default to
the shared values above. A subnetwork requires a network. If both network
settings are omitted, `gcloud` uses the Google Cloud default network
configuration.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_gke_autopilot_project_id` | `gcp_vm_project_id`, otherwise `GCP_PROJECT_ID` | Google Cloud project in which the cluster is created. |
| `gcp_gke_autopilot_auth_kind` | `application` | Credential type used to enable the Kubernetes Engine API. |
| `gcp_gke_autopilot_region` | `GCP_REGION` or `europe-west3` | Default region; an entry can override it with `region`. |
| `gcp_gke_autopilot_release_channel` | `regular` | Default release channel; an entry can override it with `release_channel`. Values are passed to `gcloud` in lowercase. |
| `gcp_gke_autopilot_enable_private_nodes` | `true` | Default for creating nodes without public IP addresses. An entry can override it with `enable_private_nodes`. |
| `gcp_gke_autopilot_network` | Empty | Default VPC network; an entry can override it with `network`. |
| `gcp_gke_autopilot_subnetwork` | Empty | Default subnet; an entry can override it with `subnetwork`. Requires a network. |
| `gcp_gke_autopilot_clusters` | One cluster named `gcp-ansible-autopilot` | Cluster entries to ensure in the project. |

Role variables use the `gcp_gke_autopilot_` prefix and can also be overridden
directly in a playbook, for example:

```yaml
roles:
  - role: gcp_gke_autopilot
    vars:
      gcp_gke_autopilot_clusters:
        - name: app-autopilot
        - name: batch-autopilot
          region: europe-west1
```

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

- The role requires the Google Cloud CLI because the installed
	`google.cloud.gcp_container_cluster` module does not expose GKE Autopilot
	configuration.
- The role creates a cluster when none with the configured name and region
	exists. It does not update the configuration of an existing Autopilot
	cluster.
- A Standard cluster with the same name and region causes the role to fail;
	choose a different cluster name instead.
- Private nodes have no direct internet egress. Configure Cloud NAT or another
	approved egress path if workloads need to access external endpoints.
- A cluster left in a non-running state by a failed creation is not deleted
	automatically. Review and remove it before rerunning the playbook.
- The role does not delete clusters.

## Run Ansible

After configuring `inventory/gke_autopilot.yml` and
`inventory/group_vars/gcp_provision.yml`, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gke_autopilot.yml
```

To validate the playbook without contacting Google Cloud, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini --syntax-check playbooks/create_gke_autopilot.yml
```

## Deletion

Cluster deletion is deliberately separate from provisioning. After confirming
that workloads and data can be removed, delete the cluster with:

```bash
gcloud container clusters delete gcp-ansible-autopilot \
	--region europe-west3 \
	--project your-project-id
```
# Ansible Role: gcp_gke_autopilot

Creates a regional Google Kubernetes Engine (GKE) Autopilot cluster.

## Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- Google Cloud CLI (`gcloud`) installed and available in `PATH`
- A Google Cloud project with billing enabled
- Permissions to enable APIs and create GKE clusters

The role enables the Kubernetes Engine API. The `gcp_project` role, included
by the repository playbook, creates the project if needed and enables the
Compute Engine API.

## Usage

Configure `gcp_vm_project_id` in `inventory/group_vars/gcp_provision.yml`, then
run the repository playbook:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gke_autopilot.yml
```

The playbook uses the `gcp_project` role before creating the cluster. To use
another cluster name or region, set the role variables in inventory or in the
playbook.

## Configuration

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_gke_autopilot_project_id` | `gcp_vm_project_id`, otherwise `GCP_PROJECT_ID` | Google Cloud project for the cluster. |
| `gcp_gke_autopilot_auth_kind` | `application` | Credential type for enabling the GKE API. |
| `gcp_gke_autopilot_region` | `GCP_REGION` or `europe-west3` | Regional cluster location. |
| `gcp_gke_autopilot_cluster_name` | `gcp-ansible-autopilot` | GKE cluster name. |
| `gcp_gke_autopilot_release_channel` | `REGULAR` | GKE release channel. |
| `gcp_gke_autopilot_network` | Empty (Google Cloud default) | Optional VPC network name. |
| `gcp_gke_autopilot_subnetwork` | Empty (Google Cloud default) | Optional subnet name; requires a network. |

Example overrides in `inventory/group_vars/gcp_provision.yml`:

```yaml
gcp_gke_autopilot_cluster_name: app-autopilot
gcp_gke_autopilot_region: europe-west3
gcp_gke_autopilot_network: app-network
gcp_gke_autopilot_subnetwork: app-europe
```

The role does not modify an existing cluster. It succeeds without changes if
the named regional cluster is already Autopilot, and fails if a Standard
cluster already uses that name and region.

## Validation

Check the playbook syntax without contacting Google Cloud:

```bash
ansible-playbook -i inventory/hosts_provision.ini --syntax-check playbooks/create_gke_autopilot.yml
```