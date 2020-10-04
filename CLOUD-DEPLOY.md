*** Cluster Setup for Google Container Engine ***

####  Install and configure local gcloud and kubectl:

Documentation: https://cloud.google.com/sdk/docs/

### Install Google Kubernetes Engine

`gcloud components install kubectl`

### Configure Google Cloud account:
```
gcloud config set account YOUR_EMAIL_ADDRESS \
&& gcloud config set project YOUR_PROJECT_ID \
&& gcloud config set compute/zone us-west1-a \
&& gcloud config set container/cluster ethereum-cluster \
&& gcloud container clusters get-credentials ethereum-cluster \
```

### Create Pay ID Cluster

```
gcloud beta container clusters create "ethereum-cluster" \
  --project "gambit-prod-ha" \
  --zone "us-central1-c" \
  --no-enable-basic-auth \
  --cluster-version "1.16.13-gke.1" \
  --release-channel "regular" \
  --machine-type "e2-small" \
  --image-type "COS" \
  --disk-type "pd-standard" \
  --disk-size "100" \
  --metadata disable-legacy-endpoints=true \
  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" \
  --num-nodes "1" \
  --enable-stackdriver-kubernetes \
  --enable-ip-alias \
  --network "projects/gambit-prod-ha/global/networks/default" \
  --subnetwork "projects/gambit-prod-ha/regions/us-central1/subnetworks/default" \
  --default-max-pods-per-node "110" \
  --no-enable-master-authorized-networks \
  --addons HorizontalPodAutoscaling,HttpLoadBalancing \
  --enable-autoupgrade \
  --enable-autorepair \
  --max-surge-upgrade 1 \
  --max-unavailable-upgrade 0 \
  --security-group="gke-security-groups@harpangell.com"
```

### Create Persistant Disk

`gcloud compute disks create --size 100GB ethereum-disk`

### Add Static IP Address

`gcloud compute addresses create ethereum-ip --global`

### Create IAM Policies... ??

`kubectl apply -f kubernetes/rbac/rbac.yml`

## ERROR: inotify

#### Open new cluster in command terminal -

`echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`

#### Commit Changes

`sysctl --system`


## Deploy Execute

```
sudo chmod +x ./scripts/build-push-deploy.sh
```

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

## Destroy Script:

```
&& kubectl delete -f cloudbuild.yaml \
&& gcloud container clusters delete example \
&& gcloud compute disks delete example-disk \
&& gcloud compute addresses delete example-ingress-ip \
```
