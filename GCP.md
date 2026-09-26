# Google Cloud Platform

## Local prerequisites

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

Create or select a Google Cloud project. You also need a billing account, the Compute Engine API, and sufficient IAM permissions.

## Billing

List the billing accounts you can use and note the desired billing account ID:

```bash
gcloud billing accounts list
```

After the project exists, link it to that billing account:

```bash
gcloud billing projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID
```

Replace `PROJECT_ID` with the value of `gcp_vm_project_id` from
`inventory/group_vars/provision.yml` and `BILLING_ACCOUNT_ID` with the ID from
the list command. Do this before provisioning the VM. The account running the
command needs permission to associate projects with the billing account
(typically `roles/billing.user` on the billing account and suitable permissions
on the project). The `gcp_project` Ansible role creates a missing project but
does not link billing automatically.

## Folder

List the folder

```bash
gcloud organizations list
```

## Projects

List the projects

```bash
gcloud projects list
```

## ssh
To connect to a VM with a private IP you need to do this:

```bash
gcloud compute ssh ansible@<hostname> \
  --project="$GCP_PROJECT_ID" \
  --zone=europe-west3-a \
  --tunnel-through-iap \
  --ssh-key-file=.ssh/gcp_linux
```
