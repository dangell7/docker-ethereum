*** Cluster Setup for Google Container Engine ***

####  Install and configure local gcloud and kubectl:

Documentation: https://cloud.google.com/sdk/docs/

### Install Google Kubernetes Engine

`gcloud components install kubectl`

### Configure Google Cloud account:
```
gcloud config set account denis@harpangell.com \
&& gcloud config set project utils-harpangell \
&& gcloud config set compute/zone us-central1-c \
&& gcloud config set container/cluster ethereum-cluster \
&& gcloud container clusters get-credentials ethereum-cluster \
&& gcloud auth configure-docker
```

### Create Ethereum Cluster

```
gcloud beta container clusters create "ethereum-cluster" \
  --project "utils-harpangell" \
  --zone "us-central1-c" \
  --no-enable-basic-auth \
  --cluster-version "1.17.14-gke.400" \
  --release-channel "regular" \
  --machine-type "e2-custom-2-2048" \
  --image-type "COS" \
  --disk-type "pd-standard" \
  --disk-size "100" \
  --metadata disable-legacy-endpoints=true \
  --service-account "compute-engine-default-service@utils-harpangell.iam.gserviceaccount.com" \
  --num-nodes "1" \
  --enable-stackdriver-kubernetes \
  --enable-ip-alias \
  --network "projects/utils-harpangell/global/networks/default" \
  --subnetwork "projects/utils-harpangell/regions/us-central1/subnetworks/default" \
  --default-max-pods-per-node "110" \
  --enable-autoscaling \
  --min-nodes "1" \
  --max-nodes "1" \
  --enable-master-authorized-networks \
  --master-authorized-networks 68.205.142.45/32 \
  --addons HorizontalPodAutoscaling,HttpLoadBalancing \
  --enable-autoupgrade \
  --enable-autorepair \
  --max-surge-upgrade 1 \
  --max-unavailable-upgrade 0 \
  --shielded-secure-boot \
  --security-group="gke-security-groups@harpangell.com"
```

### Create Persistant Disk

`gcloud compute disks create --size 100GB harpangell-disk`

### Add Static IP Address

`gcloud compute addresses create harpangell-ip --global`

### Create IAM Policies

`kubectl apply -f cloudrbac.yaml`

## Deploy Script

```
docker-compose \
-f docker-compose.yml \
-f docker-compose.gcp.yml \
build \
&& docker-compose \
-f docker-compose.yml \
-f docker-compose.gcp.yml \
push \
&& gcloud builds submit
```

### Create Cert & Ingress

`kubectl apply -f cloudcert.yaml`

`kubectl create secret generic node1-ethereum-key --from-file ./nodes/node1/password.txt`

`kubectl create secret generic node2-ethereum-key --from-file ./nodes/node2/password.txt`

## Destroy Script:

```
&& kubectl delete -f cloudbuild.yaml \
&& gcloud container clusters delete harpangell-cluster \
&& gcloud compute disks delete harpangell-disk \
&& gcloud compute addresses delete harpangell-ip \
```
