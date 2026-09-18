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

Verify that `gcloud`, `ansible`, and `ansible-galaxy` are available. Install the Python and Ansible dependencies:

```bash
python3 -m venv .myenv
source .myenv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

Create or select a Google Cloud project. You also need a billing account, the Compute Engine API, and sufficient IAM permissions. For a personal learning account, `roles/compute.admin` is typically sufficient; use a narrower role in production.

### 2. Create the VM

Set the required variable and start with this small, low-cost example:

```bash
export GCP_PROJECT_ID="your-project-id"
ansible-playbook playbooks/create_gcp_vm.yml
```

You can override these variables if needed:

```bash
export GCP_ZONE="europe-west3-a"
export GCP_MACHINE_TYPE="e2-micro"
export GCP_IMAGE_PROJECT="debian-cloud"
export GCP_IMAGE_FAMILY="debian-12"
ansible-playbook playbooks/create_gcp_vm.yml
```

The default configuration creates a VPC network, a subnet, an SSH firewall rule, and a VM with an external IPv4 address. For the first run, use:

```bash
ansible-playbook playbooks/create_gcp_vm.yml --check
```

`--check` does not fully replace an API permission check. After the actual run, inspect the VM with `gcloud compute instances describe gcp-ansible-vm --zone "$GCP_ZONE" --project "$GCP_PROJECT_ID"`.

### 3. Delete the VM

Deletion uses a separate command so that normal provisioning never removes resources:

```bash
gcloud compute instances delete gcp-ansible-vm --zone "$GCP_ZONE" --project "$GCP_PROJECT_ID"
```

The network and firewall rule remain in place and can be removed separately through the Google Cloud console or `gcloud`.
