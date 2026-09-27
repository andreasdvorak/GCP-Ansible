# gcp_storage

Creates a Google Cloud Storage bucket.

## Requirements

- The `google.cloud` Ansible collection
- Google Cloud Application Default Credentials
- A globally unique bucket name for every bucket definition
- A `cost_center` and `function` value for every bucket definition

Install the collection from the repository root with:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Configuration

Bucket definitions are kept independently from host variables in
`inventory/storage_buckets.yml`:

```yaml
gcp_storage_buckets:
  - name: your-globally-unique-bucket-name
    cost_center: 100
    function: mytest
    location: europe-west3
```

The role applies these labels to every bucket by default:

```yaml
gcp_storage_labels:
  managed-by: ansible
  cost_center: "<bucket.cost_center>"
  function: "<bucket.function>"
```

Override the role variables when needed:

| Variable | Default | Description |
| --- | --- | --- |
| `gcp_storage_project_id` | `gcp_vm_project_id` | Google Cloud project ID |
| `gcp_storage_auth_kind` | `application` | Authentication method |
| `gcp_storage_location` | `GCP_REGION` or `europe-west3` | Bucket location |
| `gcp_storage_storage_class` | `STANDARD` | Bucket storage class |
| `gcp_storage_buckets` | `[]` | List of bucket definitions |

The name of the storage bucket muss be unique in the whole GCP.

## Usage

Add one or more bucket definitions to `inventory/storage_buckets.yml` and run:

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_storage.yml
```