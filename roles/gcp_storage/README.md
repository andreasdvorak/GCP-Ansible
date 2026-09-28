# Ansible Role: gcp_storage

Creates Google Cloud Storage buckets and enables the Cloud Storage API in the
configured project.

### Features

- Creates one or more Cloud Storage buckets from inventory definitions.
- Applies required management, cost center, and function labels, with support
  for additional custom labels.
- Allows per-bucket project, location, and storage class overrides.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role has no Ansible role dependencies. |

### Optional Roles

| Role | Required | Purpose |
| --- | --- | --- |
| `gcp_project` | No | Creates the project before bucket provisioning. The storage role enables the Cloud Storage API itself. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- An existing Google Cloud project, or the `gcp_project` role earlier in the
  play
- Permissions to enable the Cloud Storage API and create buckets
- A globally unique bucket name for every bucket definition
- A `cost_center` and `function` value for every bucket definition

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

Add one or more bucket definitions to `inventory/storage_buckets.yml`. The
repository playbook loads that file and creates the project first when needed:

```yaml
- name: Provision Google Cloud Storage buckets
  hosts: gcp_provision
  gather_facts: false
  vars_files:
    - ../inventory/storage_buckets.yml
  roles:
    - role: gcp_project
    - role: gcp_storage
```

The repository entry point is `playbooks/create_gcp_storage.yml`. Provisioning
configuration is in `inventory/group_vars/gcp_provision.yml`; bucket definitions
are kept separately in `inventory/storage_buckets.yml`.

## Configuration

The recommended bucket configuration file is `inventory/storage_buckets.yml`.
Each entry in `gcp_storage_buckets` requires a globally unique `name`, a
`cost_center`, and a `function`. The other bucket properties are optional:

```yaml
gcp_storage_buckets:
  - name: your-globally-unique-bucket-name
    cost_center: 100
    function: mytest
    location: europe-west3
    storage_class: STANDARD
    labels:
      environment: dev
```

The role always applies `managed-by: ansible` and the configured `cost_center`
and `function` labels. Additional labels can be provided in the bucket's
`labels` mapping; the role-managed labels take precedence if keys overlap.

The role defaults `gcp_storage_project_id` to `gcp_vm_project_id`, as configured
by the repository inventory. For standalone use, define
`gcp_storage_project_id` directly. `GCP_REGION` provides the default bucket
location.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_storage_project_id` | `gcp_vm_project_id` | Default Google Cloud project for buckets. A bucket's `project_id` can override it. |
| `gcp_storage_auth_kind` | `application` | Credential type used by the Google Cloud modules. |
| `gcp_storage_location` | `GCP_REGION` or `europe-west3` | Default bucket location. A bucket's `location` can override it. |
| `gcp_storage_storage_class` | `STANDARD` | Default bucket storage class. A bucket's `storage_class` can override it. |
| `gcp_storage_buckets` | `[]` | List of bucket definitions. Each entry requires `name`, `cost_center`, and `function`. |

Role variables use the `gcp_storage_` prefix and can also be overridden in a
playbook, for example:

```yaml
roles:
  - role: gcp_storage
    vars:
      gcp_storage_location: europe-west3
      gcp_storage_storage_class: STANDARD
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

- Cloud Storage bucket names must be globally unique.
- Every bucket definition must include `cost_center` and `function` values for
  labels.
- A bucket's location cannot be changed after creation.
- The role creates buckets but does not delete them.

## Run Ansible

After configuring `inventory/group_vars/gcp_provision.yml` and
`inventory/storage_buckets.yml`, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_storage.yml
```

To validate the playbook without contacting Google Cloud, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini --syntax-check playbooks/create_gcp_storage.yml
```

## Deletion

Bucket deletion is deliberately separate from provisioning. Delete a bucket
with `gcloud` after confirming that its contents are no longer needed:

```bash
gcloud storage buckets delete gs://your-globally-unique-bucket-name \
  --project your-project-id
```