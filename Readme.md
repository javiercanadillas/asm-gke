# Introduction

This repo contains all the necessary artifacts to go from zero to demo Anthos Service Mesh 1.8.1.

`asm_gke` automates the deployment and destruction of a Anthos Service Mesh GKE enabled cluster in GCP. The script is designed to work witht the [ASM 1.8.1 installation script](https://cloud.google.com/service-mesh/docs/scripted-install/reference) provided by Google Cloud.

The script is designed to work with GCP's Cloud Shell. You will need editor permissions on a GCP project and, if you're a googler, GCE enforcer disabled.

# Setting up the environment

Open Cloud Shell, and make sure your active project is the one you want to deploy everything in:

```bash
gcloud config set project <your project ID>
```

Once that's done, run the main script:

```bash
./asm_gke install
```

That's it. You should have a cluster running with Anthos Service Mesh enabled on it and the sample Istio application [Bookinfo](https://istio.io/latest/docs/examples/bookinfo/) deployed. For step-by-step instructions on demoing ASM, read on.

# Demoing ASM

# Tearing down the environment
To tear down the environment, run:

```bash
./asm_gke destroy
```
