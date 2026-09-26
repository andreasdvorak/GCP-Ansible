# Ansible Role: gcp_project

Creates a Google Cloud project when it does not already exist and enables the
Compute Engine API. The project ID is taken from `gcp_vm_project_id` so the
following VM and labels roles target the same project.

## Dependencies

### Required Roles

| Role | Required | Purpose |
| --- | --- | --- |
| None | - | The role uses the `google.cloud` collection directly. |

### External Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- Permissions to create projects, attach them to the configured organization
  (if set), and enable services
- The Service Usage API must be available to enable Compute Engine
- A billing account must be linked to the project before creating billable
  resources such as Compute Engine VMs

## Configuration

Set the desired, globally unique project ID in `gcp_vm_project_id`, for example
in `inventory/group_vars/provision.yml`. The role does not invent an ID when
this setting is missing because generated IDs would not be stable between runs.

| Role variable | Default | Description |
| --- | --- | --- |
| `gcp_project_id` | `gcp_vm_project_id` | Desired Google Cloud project ID. |
| `gcp_project_display_name` | Unset | Optional display name for a newly created project; if unset, existing project names are left unchanged. Must be 4-30 characters when set. |
| `gcp_project_parent_organization_id` | Empty | Optional organization ID under which to create the project. |
| `gcp_project_auth_kind` | `application` | Credential type used by the Google Cloud modules. |

If the project belongs to an organization, set its ID as well:

```yaml
gcp_project_parent_organization_id: "123456789012"
```

The project ID must be 6-30 lowercase letters, digits, or hyphens, start with a
letter, and not end with a hyphen. It must also be globally unique in Google
Cloud.

## Usage

The repository's `playbooks/create_gcp_vm.yml` runs this role before creating
the VM. To create the project independently, include `gcp_project` in a play
that targets the provisioning inventory group.

## Known Limitations

- The role does not create or link a billing account. Link billing before VM
  provisioning.
- The caller must already have permission to create projects and enable APIs.
- When an organization requires projects to be created under a parent, configure
  `gcp_project_parent_organization_id`.

## Run Ansible

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_vm.yml
```