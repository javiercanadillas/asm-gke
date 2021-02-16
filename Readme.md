# Introduction

This repo contains all the necessary artifacts to go from zero to demo the latest version of Anthos Service Mesh, ASM 1.8.2.

`asm_gke` automates the deployment and destruction of a Anthos Service Mesh GKE enabled cluster in GCP. The script is designed to work witht the [ASM 1.8.x installation script](https://cloud.google.com/service-mesh/docs/scripted-install/reference) provided by Google Cloud.

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

## But I want to run this from my local machine
The script is not yet ready to deploy the istioclt binaries in your local machine, whatever your operating system may be. It's designed and teste only for Google Cloud Shell. However, you can get a close experience by connecting to Cloud Shell from your local shell using the GCP SDK.

For illustration purposes, let's say you have a Mac. Install Google Cloud SDK, so you have the gcloud tool installed in your system. Now, let's imagine you use iTerm2. Open a new shell session and do, authenticate gcloud with the Google account you'll be using in your GCP project and do:

```bash
gcloud cloud-shell ssh --authorize-session
```

Voilà, you now have a Cloud Shell session opened from your local terminal. If you also want to manipulate remote files with an IDE, say VSCode, you can also open a new local shell session and do:

```
# Creates a mount point for Cloud Shell filesystem
mkdir -p ~/cloudshell
# Gets the command to mount the Cloud Shell remote filesystem on cloudshell/ and executes it
$(cloud cloud-shell get-mount-command cloudshell)
# Now change to the mount point and open VSCode from there
cd cloudshell
code .
```

You have now your local IDE editing your remote files in Cloud Shell, so you can take advantage of whatever plugins or workflows you may have enabled in your IDE.

# Demoing ASM

The `asm_gke` script has deployed Istio's Bookinfo application for you. Read the [introduction at Istio's website](https://istio.io/latest/docs/examples/bookinfo/) to understand the basics of what you've just deployed. The application microservices architecture looks like this:

![Bookinfo](https://istio.io/latest/docs/examples/bookinfo/withistio.svg)

Having a polyglot application (with microservices written in different languages), although a reflection of real life, is typically a pain in the ass for a demoer to fully understand the deployment details in Kubernetes. But in this case, we're talking about demoing Istio and it's ability to abstract away implementation details (like security and communications in this case), so it's quite relevant to have it this way.

## Traffic Management

The application as it is should not be accesible from outside the cluster. This is like so because there's no service exposed outside the cluster specific for the application.




First, let's confirm that the application is accesible from outside the cluster:

```bash
GATEWAY_IP=$(./asm_gke get-gw-ip)
curl -s "https://GATEWAY_IP
```

# Tearing down the environment
To tear down the environment and restore your project the way it was before running the script, run:

```bash
./asm_gke destroy
```
