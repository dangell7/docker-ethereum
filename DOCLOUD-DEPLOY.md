*** Cluster Setup for Digital Ocean Kubernetes ***

####  Install and configure local doctl and kubectl:

Documentation: https://github.com/digitalocean/doctl

### Install DO CTL

`brew install doctl`

### Install Kube CTL

`brew install install kubectl`

### Configure Digital Ocean account:

Documentation: https://cloud.digitalocean.com/account/api/tokens

```
doctl auth init
doctl kubernetes cluster kubeconfig save 63a4ebae-a648-44b2-93d8-c210083d3326
```

### Install 1-click apps

```
Kubernetes Monitoring Stack
NGINX Ingress Controller
```

### Create IAM Policies

`kubectl apply -f cloudrbac.yaml`

### Login to DO Docker Container Registry

`doctl registry login`

## Deploy Script

```
docker-compose \
-f docker-compose.yml \
-f docker-compose.do.yml \
build \
&& docker-compose \
-f docker-compose.yml \
-f docker-compose.do.yml \
push \
&& kubectl apply -f doclouddeploy.yaml
```

### Deploy Cert & Ingress

`kubectl apply -f docloudcert.yaml`
