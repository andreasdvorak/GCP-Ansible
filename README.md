## Google Cloud with Ansible

This repository creates a services in an existing Google Cloud project. The Google Cloud API and Compute Engine API must be enabled in the project.

For Ansible information read this: [Ansible](Ansible.md)

For GCP information read this: [GCP](GCP.md)


### Project structure

The VM provisioning is implemented in the `gcp_vm` role. See [roles/gcp_vm/README.md](roles/gcp_vm/README.md) for role configuration, usage, and resource management.
