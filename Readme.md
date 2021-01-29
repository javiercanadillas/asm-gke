# Introduction

This repo contains all the necessary artifacts to go from zero to demo Anthos Service Mesh 1.8.1.

`asm_gke` automates the deployment and destruction of a Anthos Service Mesh GKE enabled cluster in GCP. The script is designed to work witht the [ASM 1.8.1 installation script](https://cloud.google.com/service-mesh/docs/scripted-install/reference) provided by Google Cloud.

The script is designed to work with GCP's Cloud Shell. You will need editor permissions on a GCP project and, if you're a googler, GCE enforcer disabled.

# Setting up the environment

Open Cloud Shell, and make sure your active project is the one you want to deploy everything in:

```bash
gcloud config set project <your project ID>
```

Once that's done, check the environment variables picked up by the script:

```bash
./asm_gke show-config
```

and when you're confidend everything looks alright run it:

```bash
./asm_gke install
```

That's it. You should have a cluster running with Anthos Service Mesh enabled on it and the sample Istio application [Bookinfo](https://istio.io/latest/docs/examples/bookinfo/) deployed.

## Getting help and options

Run:
```bash
./asm_gke --help
```

For step-by-step instructions on demoing ASM, read on.

# Demoing ASM

Now you have Istio's Bookinfo application deployed. Read the [introduction at Istio's website](https://istio.io/latest/docs/examples/bookinfo/) to understand the basics of what you've just deployed. The application microservices architecture looks like this:

![Bookinfo](https://istio.io/latest/docs/examples/bookinfo/noistio.svg)

Having a polyglot application (with microservices written in different languages), although a reflection of real life, is typically a pain in the ass for a demoer to fully understand the deployment details in Kubernetes. But in this case, we're talking about demoing Istio and it's ability to abstract away implementation details (like security and communications in this case), so it's quite relevant to have it this way.



# Tearing down the environment
To tear down the environment and restore your project the way it was before running the script, run:

```bash
./asm_gke destroy
```
