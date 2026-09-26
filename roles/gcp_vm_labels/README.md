# gcp_vm_labels

Applies the desired GCP labels to an existing Compute Engine VM. The role only
updates labels; it does not create or otherwise reconfigure the VM.

### Features

- Verifies that the target VM exists before changing it.
- Applies the configured label map to the VM without changing its other
  configuration.
- Can run independently or after the `gcp_vm` role in the same play.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role has no Ansible role dependencies. |

### Optional Roles

| Role | Required | Purpose |
| --- | --- | --- |
| `gcp_vm` | No | Can provision the VM earlier in the same play; it is not required to update labels on an existing VM. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- An existing Compute Engine VM in a Google Cloud project with the Compute
  Engine API enabled
- Permissions to retrieve the VM and update its labels

Install the collection from the repository root with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Supported Platforms

| Platform | Version |
| --- | --- |
| Existing Compute Engine VM | Any guest OS supported by Compute Engine; the role only updates cloud-side labels. |

## Supported Clouds

| Cloud Platform | Supported |
| --- | --- |
| GCP | Yes |

## Usage

Apply labels without running VM provisioning:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/update_gcp_vm_labels.yml
```

The repository's provisioning playbook also runs this role after `gcp_vm`:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_vm.yml
```

## Configuration

Set `gcp_vm_labels_desired` in the target host's variables. The map is
authoritative: exactly these labels are applied, so labels not present in the
map are removed.

```yaml
gcp_vm_labels_desired:
  managed-by: ansible
  function: "{{ gcp_vm_function }}"
  cost_center: "{{ cost_center | string }}"
  hostenvironment: "{{ hostenvironment }}"
```

The role uses these variables to identify and authenticate to the VM:

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_vm_labels_desired` | `managed-by: ansible`, `function: <inventory host name>` | Desired VM labels. This map replaces the VM's current labels. |
| `gcp_vm_project_id` | Required | Project containing the VM. |
| `gcp_vm_labels_instance_name` | `gcp_vm_instance_name` if set; otherwise inventory host name | Name of the existing Compute Engine VM. |
| `gcp_vm_zone` | Required | Zone containing the VM. |
| `gcp_vm_auth_kind` | Required | Credential type used by the Google Cloud modules; use `application` with Application Default Credentials. |

The `gcp_vm_labels` role does not load defaults from `gcp_vm` when run by
itself. The repository's `inventory/group_vars/provision.yml` sets the project
ID, but a standalone labels run must also define the VM name, zone, and auth
kind in inventory or play variables. The VM name defaults to the inventory host
name, matching the basename of its `host_vars` file. For example, add these
shared settings to `inventory/group_vars/provision.yml` (adjust the project,
zone, and auth kind):

```yaml
gcp_vm_project_id: "your-project-id"
gcp_vm_zone: europe-west3-a
gcp_vm_auth_kind: application
```

When both roles are included in the same play, `gcp_vm` supplies its defaults
for these target settings. Label values should be strings; cast non-string
inventory values, such as `cost_center`, with `| string`.

## Encrypted Data / Ansible Vault

The role does not require encrypted variables or Ansible Vault. Credentials are
read through Application Default Credentials; protect any sensitive inventory
values with Ansible Vault if your environment requires it.

## Tags

Available tags:

| Tag | Purpose |
| --- | --- |
| None | No role-specific Ansible task tags are defined. |

## Known Limitations

- The target VM must already exist; this role does not create, delete, or
  otherwise reconfigure it.
- The desired label map is authoritative and removes existing labels omitted
  from the map.
- A standalone run must define all target variables (`gcp_vm_project_id`,
  `gcp_vm_zone`, and `gcp_vm_auth_kind`). The instance name defaults to
  `inventory_hostname`.

## Run Ansible

After configuring the target variables and labels, run:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/update_gcp_vm_labels.yml
```

The role first verifies that the VM exists, then updates its labels through the
Google Cloud collection.

## Deletion

Not applicable. This role only updates labels and does not delete the VM or
other Google Cloud resources.