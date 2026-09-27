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
```

Choose one stable quota project for API usage and quotas. It can be a dedicated
administration project and does not have to be the project Ansible is currently
creating or configuring. The quota project is stored in the Application Default
Credentials, so set it once per credentials/environment, not once per target
project. The identity used by Ansible needs `serviceusage.services.use` on it.

Enable the APIs used for the API calls in the quota project and set the ADC quota
project:

```bash
export GCP_QUOTA_PROJECT_ID="YOUR_QUOTA_PROJECT_ID"
gcloud services enable cloudresourcemanager.googleapis.com serviceusage.googleapis.com \
  --project="$GCP_QUOTA_PROJECT_ID"
gcloud auth application-default set-quota-project "$GCP_QUOTA_PROJECT_ID"
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

## APIs in target projects

The Cloud Resource Manager API must be enabled in the quota project above; it
does not need to be enabled separately every time the target project changes.
The target project still needs the APIs for the resources Ansible manages, for
example Compute Engine. The `gcp_project` role enables the Compute Engine API.

To inspect APIs enabled in a target project:
```bash
gcloud services list --enabled --project=TARGET_PROJECT_ID
```

`gcloud config set project` changes the default project for `gcloud` commands;
it does not change the quota project stored in ADC.
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
