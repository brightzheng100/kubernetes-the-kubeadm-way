# Prerequisites

## Google Cloud Platform SDK

### Install the Google Cloud SDK

Follow the Google Cloud SDK [documentation](https://cloud.google.com/sdk/) to install and configure the `gcloud` command line utility.

Verify the Google Cloud SDK version is 218.0.0 or higher:

```sh
gcloud version
```

> Output

```
Google Cloud SDK 241.0.0
bq 2.0.43
core 2019.04.02
gsutil 4.38
```

### Set a Default Compute Region and Zone

This tutorial assumes a default compute region and zone have been configured.

If you are using the `gcloud` command-line tool for the first time `init` is the easiest way to do this:

```sh
gcloud init
```

Otherwise set a default compute region. For example:

```sh
gcloud config set compute/region us-west1
```

Set a default compute zone. For example:

```sh
gcloud config set compute/zone us-west1-b
```

> Use the `gcloud compute zones list` command to view additional regions and zones.


Next: [02-Provisioning Infrastructure](02-infrastructure.md)
