## Google Cloud with Ansible

This repository creates a Google Cloud project when needed and provisions
services in it. The project ID is configured in
`inventory/group_vars/provision.yml`; a billing account must be linked before
creating billable resources such as Compute Engine VMs.

For Ansible information read this: [Ansible](Ansible.md)

For GCP information read this: [GCP](GCP.md)


### Project structure

Project creation and Compute Engine API activation are implemented in the
`gcp_project` role. VM provisioning is implemented in the `gcp_vm` role. See
[roles/gcp_project/README.md](roles/gcp_project/README.md) and
[roles/gcp_vm/README.md](roles/gcp_vm/README.md) for configuration and usage.

Provisioning runs against `inventory/hosts_provision.ini` and connects locally to
Google Cloud. Runtime playbooks use `inventory/hosts_runtime.ini`, which is also
the default inventory in `ansible.cfg`. VM labels can be changed independently
with the `gcp_vm_labels` role and `playbooks/update_gcp_vm_labels.yml`.

```bash
ansible-playbook -i inventory/hosts_provision.ini playbooks/create_gcp_vm.yml
ansible-playbook -i inventory/hosts_provision.ini playbooks/update_gcp_vm_labels.yml
```
