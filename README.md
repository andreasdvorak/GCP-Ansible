## Google Cloud VM with Ansible

This repository creates a Compute Engine VM in an existing Google Cloud project. The Google Cloud API and Compute Engine API must be enabled in the project.

### 1. Local prerequisites

First, install the Google Cloud CLI by following the official installation instructions for Linux.

[Installation Guide](https://docs.cloud.google.com/sdk/docs/install-sdk?hl=de#linux)

Then verify it:

```bash
gcloud --version
```

Next, authenticate with local Application Default Credentials:

```bash
gcloud auth application-default login
gcloud auth application-default set-quota-project YOUR_PROJECT_ID
```

Check the login
```bash
gcloud auth list
gcloud auth application-default print-access-token
```

Verify that `gcloud`, `ansible`, and `ansible-galaxy` are available. Install the Python and Ansible dependencies:

```bash
python3 -m venv .myenv
source .myenv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

Create or select a Google Cloud project. You also need a billing account, the Compute Engine API, and sufficient IAM permissions. For a personal learning account, `roles/compute.admin` is typically sufficient; use a narrower role in production.

### 2. Project structure

The VM provisioning is implemented in the `gcp_vm` role. See [roles/gcp_vm/README.md](roles/gcp_vm/README.md) for role configuration, usage, and resource management.
