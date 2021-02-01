# Common 

This is my draft script for writing the bash script automating the demo foundations. It'll be deleted as soon as instructions for demoing ASM are complete.

Do all this in `~/code`.

## Export variables
```bash
export PROJECT_ID=javiercm-main-demos
export CLUSTER_ZONE=europe-west1-b
export CLUSTER_LOCATION=$CLUSTER_ZONE
export CLUSTER_NAME=gke-asm
export DIR_PATH="~/code/anthos"
export MEMBERSHIP_NAME=gke-asm-connect
```

# Install

## Go to working directory
```bash
cd ~/code
```

## Create cluster

```bash
gcloud config set compute/zone ${CLUSTER_ZONE}
  gcloud beta container clusters create ${CLUSTER_NAME} \
      --machine-type=n1-standard-4 \
      --num-nodes=4 \
      --enable-stackdriver-kubernetes \
      --subnetwork=default \
      --release-channel=regular
```

## Download the script
```bash
curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.8 > install_asm
chmod +x install_asm
```

## Update cluster labels
```bash
gcloud container clusters update $CLUSTER_NAME \
    --update-labels \
    mesh_id=proj-912971218747,asmv=1-8-1-asm-5
```

## Enable workload identity
```bash
gcloud container clusters update $CLUSTER_NAME \
  --workload-pool=$PROJECT_ID.svc.id.goog
  ```

## Register cluster to the environ
```bash
gcloud beta container hub memberships register $MEMBERSHIP_NAME \
   --gke-cluster=$CLUSTER_ZONE/$CLUSTER_NAME \
   --enable-workload-identity
```

## Make myself cluster admin
```bash
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```

## Validate
```bash
mkdir -p $DIR_PATH
./install_asm \
  --project_id $PROJECT_ID \
  --cluster_name $CLUSTER_NAME \
  --cluster_location $CLUSTER_LOCATION \
  --mode install \
  --output_dir $DIR_PATH \
  --only_validate
```

## Install
```
./install_asm \
  --project_id $PROJECT_ID \
  --cluster_name $CLUSTER_NAME \
  --cluster_location $CLUSTER_LOCATION \
  --mode install \
  --output_dir $DIR_PATH \
  --enable_all
```

## Enable injection
```bash
kubectl label namespace default istio-injection- istio.io/rev=asm-181-5 --overwrite
```

# Deploy Bookinfo

## Download the installation file
This is only required if you forgot to include the `--output-dir` option in the ASM installation script.

```bash
curl -LO https://storage.googleapis.com/gke-release/asm/istio-1.8.1-asm.5-linux-amd64.tar.gz
tar xzf istio-1.8.1-asm.5-linux-amd64.tar.gz
```

## Deploy application pods
```bash
cd istio-1.8.1-asm.5
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

## Configure Istio Ingress gateway for the application
```bash
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

### Verify Bookinfo deployments
```bash
kubectl get services
````

The output should be:
```text
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.3.249.198   <none>        9080/TCP   10h
kubernetes    ClusterIP   10.3.240.1     <none>        443/TCP    10h
productpage   ClusterIP   10.3.249.23    <none>        9080/TCP   10h
ratings       ClusterIP   10.3.243.97    <none>        9080/TCP   10h
reviews       ClusterIP   10.3.253.5     <none>        9080/TCP   10h
```

```bash
kubectl get pods
```

The output should be:
```text
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-5974b67c8-gc84s        2/2     Running   0          10h
productpage-v1-64794f5db4-mqfr2   2/2     Running   0          10h
ratings-v1-c6cdf8d98-t2j6p        2/2     Running   0          10h
reviews-v1-7f6558b974-562lb       2/2     Running   0          10h
reviews-v2-6cb6ccd848-jl4ww       2/2     Running   0          10h
reviews-v3-cc56b578-pckgz         2/2     Running   0          10h
```

## Confirm that Bookinfo application is running
Do it by sending a curl request to it from some pod within the cluster (ratings, for example):
```bash
kubectl exec -it $(kubectl get pod -l app=ratings \
    -o jsonpath='{.items[0].metadata.name}') \
    -c ratings -- curl productpage:9080/productpage | grep -o  "<title>.*</title>"
```

The output should be:
```text
<title>Simple Bookstore App</title>
```

## Get the external IP of the Istio IngressGateway
```bash
GATEWAY_URL=$(kubectl get svc istio-ingressgateway -n istio-system | awk '{if (NR!=1) {print $4}}')
```
# Traffic Management with ASM

## Configure cluster access for kubectl
```bash
export CLUSTER_NAME=gke-asm
export CLUSTER_ZONE=europe-west1-b
```

```bash
export GCLOUD_PROJECT=$(gcloud config get-value project)
```

```bash
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
```

## Save Gateway IP
```bash
export GATEWAY_URL=$(kubectl get svc istio-ingressgateway -o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n istio-system)
echo The gateway address is $GATEWAY_URL
```

## Generate some traffic
```bash
siege http://${GATEWAY_URL}/productpage
```

## Prepare for using additional sample configurations
```bash
cd ~/code/istio-*
export PATH=$PWD/bin:$PATH
```

## Apply default destination rules, for all available versions

Examine the definition of the destination rules:
```bash
vim samples/bookinfo/networking/destination-rule-all.yaml
```

Apply a config that defines 4 `DestinationRule` resources, 1 for each service:
```bash
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

Output:
```text
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
```

Check that the 4 `DestinationRule` resources were defined:
```bash
kubectl get destinationrules
```

```text
NAME          HOST          AGE
details       details       1m
productpage   productpage   1m
ratings       ratings       1m
reviews       reviews       1m
```

Review the details of the destination rules:
```bash
kubectl get destinationrules -o yaml
```

**Go back to QKL**

## Apply virtual services so they route by default to only one version
Apply a config that defines 4 `VirtualServices` resources, 1 for each service:
```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```
Output:
```text
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
```

Check the 4 routes, `VirtualServices` resources were defined:
```bash
kubectl get virtualservices
```

Output:
```text
NAME          GATEWAYS             HOSTS
bookinfo      [bookinfo-gateway]   [*]
details                            [details]
productpage                        [productpage]
ratings                            [ratings]
reviews                            [reviews]
```

In Cloud Shell, get the externalIP of the ingress gateway:


## Toute to a specific version of a service based on user identity

Apply the config that defines 1 virtual service resource:
```bash
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

Confirm the rule is created:
```bash
kubectl get virtualservice reviews
```

**Go back to QKL**

Start new siege session generating only 20% of the traffic of the first, but with all requests authenticated as jason:
```bash
curl -c cookies.txt -F "username=jason" -L -X \
    POST http://$GATEWAY_URL/login
cookie_info=$(grep -Eo "session.*" ./cookies.txt)
cookie_name=$(echo $cookie_info | cut -d' ' -f1)
cookie_value=$(echo $cookie_info | cut -d' ' -f2)
siege -c 5 http://$GATEWAY_URL/productpage \
    --header "Cookie: $cookie_name=$cookie_value"
```

Clean up the task:
```bash
kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
``` 

# Destroy

Todo.

# Reporting the `GKE_CLUSTER_URL` error on ASM 1.8.1 installation

```bash
cd ~/code/istio-1.8.1-asm.5
./bin/asmctl validate
```

```text
[asmctl version 0.4.0]
Using Kubernetes context: javiercm-main-demos_europe-west1-b_gke-asm
To change the context, use the --context flag
Validating enabled APIs
OK
Validating ingressgateway configuration
Missing configuration value GKE_CLUSTER_URL with value https://container.googleapis.com/v1/projects/javiercm-main-demos/locations/europe-west1-b/clusters/gke-asm.
Please set the following environment variables and try the installation instructions again:
export PROJECT_ID=javiercm-main-demos
export PROJECT_NUMBER=912971218747
export CLUSTER_ZONE=europe-west1-b
export CLUSTER_NAME=gke-asm
export IDNS=javiercm-main-demos.svc.id.goog
If you need further assistance, please run the following from your Istio repository and send the output file, istio-dump.tar.gz, to the ASM team (asm-support@google.com):
./tools/kubernetes_dump.sh -z
Error: could not validate ingressgateway configuration: found misconfiguration in Ingress Gateway pod.
Please fix the issue and try to validate again
```