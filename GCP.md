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
