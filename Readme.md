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

## Checking the deployment

Let's check what we've got deployed in the default namespace by getting the services:

```bash
kubectl get svc
```
```text
details       ClusterIP   10.3.249.116   <none>        9080/TCP   109m
kubernetes    ClusterIP   10.3.240.1     <none>        443/TCP    117m
productpage   ClusterIP   10.3.244.28    <none>        9080/TCP   109m
ratings       ClusterIP   10.3.241.215   <none>        9080/TCP   109m
reviews       ClusterIP   10.3.247.53    <none>        9080/TCP   109m
```

(If the previous command throws an authentication error because you're Cloud Shell VM stopped, just grab the cluster credentials again with `gcloud container clusters get-credentials gke-asm --zone europe-west1-b`)

Let's have a look at the pods that actually implement the service:

```bash
kubectl get pods
```
```text
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-558b8b4b76-kg24c       2/2     Running   0          15h
productpage-v1-6987489c74-sd7xv   2/2     Running   0          15h
ratings-v1-7dc98c7588-dfr5w       2/2     Running   0          15h
reviews-v1-7f99cc4496-xcn2f       2/2     Running   0          15h
reviews-v2-7d79d5bd5d-x26ds       2/2     Running   0          15h
reviews-v3-7dbcdcbc56-twlh2       2/2     Running   0          15h
```

Each pod has two containers running, one for implementing the service logic and the other one for the ASM proxy sidecar that has been injected automatically when deploying the application. We can see that by inspecting any of the services and having a look a the `Containers` attribute:

```bash
kubectl describe pod productpage
```
```text
[...]
Containers:
  productpage:
    Container ID:   docker://b6929b11f06079328f9c379d2e54b9849245d232aab9d18c412d09fc4f58eda9
    Image:          docker.io/istio/examples-bookinfo-productpage-v1:1.16.2
[...]
  istio-proxy:
    Container ID:  docker://1b10430bf7f1576b77b6248876f88358253f8f879935356d27368828b42f1996
    Image:         gcr.io/gke-release/asm/proxyv2:1.8.3-asm.2
    Image ID:      docker-pullable://gcr.io/gke-release/asm/proxyv2@sha256:d3c55c913888d4d50d3f5e6f50461af14592327bc40078a67e6529ebc935bf0f
[...]
```

Let's now test that Bookinfo is running 


## An overview of the ASM dashboard



## Traffic Management

**The application as it is should not be accesible from outside the cluster**. This is like so because there's no service exposed outside the cluster specific for the application. You can check that by asking for the services in the cluster:

```bash
kubectl get svc
```

The output should be as follows:
```text
details       ClusterIP   10.3.249.116   <none>        9080/TCP   109m
kubernetes    ClusterIP   10.3.240.1     <none>        443/TCP    117m
productpage   ClusterIP   10.3.244.28    <none>        9080/TCP   109m
ratings       ClusterIP   10.3.241.215   <none>        9080/TCP   109m
reviews       ClusterIP   10.3.247.53    <none>        9080/TCP   109m
```

First, let's confirm that the application is now accesible from outside the cluster:

```bash
GATEWAY_IP=$(./asm_gke get-gw-ip)
curl -s "https://GATEWAY_IP
```

# Tearing down the environment
To tear down the environment and restore your project the way it was before running the script, run:

```bash
./asm_gke destroy
```
