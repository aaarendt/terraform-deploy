# Google Cloud VM Example

Deploy a network and a GKE cluster on Google Cloud via
Terraform. This infrastructure will host a deployment of
[`dask-gateway`](https://gateway.dask.org/).

## Setup

Download / Configure the following

- [Terraform](https://www.terraform.io/downloads.html)
- gcloud:
  - [Install](https://cloud.google.com/sdk/docs/install)
  - [Configure](https://cloud.google.com/sdk/docs/initializing)
  - Create and download a
  [service account key](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) 
- [Helm](https://helm.sh/docs/intro/quickstart/)

## Deployment

Input variables into `your-cluster.tfvars` if you want
options that differ from the defaults in `variables.tf`.
Using these variables will require adding the following flag
to all `terraform` commands: `--var-file=your-cluster.tfvars`.

In particular, you will need to supply the filepath of your
service account key to the `credential_file` input variable.

And yes, you need both the `google` and the `google-beta`
providers.

Once you are ready to deploy, you can look at the plan with:

```
terraform plan --var-file=your-cluster.tfvars
```

Deploy the network and cluster with:

```
terraform apply --var-file=your-cluster.tfvars
```

## Install `dask-gateway`

To install `dask-gateway` onto the cluster, there are a few
steps:
- Make sure Helm is installed (listed above)
- Update your `kubeconfig` file with:
```
gcloud container clusters get-credentials <your-cluster-name> --region <your-region>
```
- Add your `gcloud` credentials to kubernetes as a secret.
This is the same key references above in "create and download
a service account key."
```
kubectl -n dask-gateway create secret generic dask-worker-sa-key --from-file <~/path/to/key/file.json>
```
- Install `dask-gateway`
```
kubectl create ns dask-gateway
helm repo add daskgateway https://dask.org/dask-gateway-helm-repo/
helm repo update
helm upgrade --install -n dask-gateway --version 0.8.0 --values dask-gateway-config.yaml dask-gateway daskgateway/dask-gateway
```

## Connecting to the Gateway

To connect to the gateway, follow instructions from the
`dask` documentation:
https://gateway.dask.org/install-kube.html#connecting-to-the-gateway

Other information for using the gateway for computations
is in the `dask` documentation as well:
https://gateway.dask.org/usage.html

## Tear Down `dask-gateway`

This must be performed before trying to tear down the
infrastructure, otherwise the `terraform destroy` command
will fail.

```
helm delete dask-gateway -n dask-gateway --purge
```

## Tear Down the Infrastructure

Remove the network and cluster with:

```
terraform destroy --var-file=your-cluster.tfvars
```