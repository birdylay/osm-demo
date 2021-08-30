# Open Service Mesh (OSM) Demo

For Advanced Cloud Architecture YCIT 020.
This repository is a subset of the OSM demo provided at https://github.com/openservicemesh/osm/tree/release-v0.9 and https://docs.openservicemesh.io/docs/getting_started/quickstart/manual_demo/

This demo utilises Helm instead of the OSM CLI.   The book microservices have been changed to deploy using one Helm chart.  The access manifest files and port fowarding scripts are pulled from the OSM git repo.

## Prerequistes

Authenticated cluster running Kubernetes v1.19 or greater
Clone repo:

    git clone https://github.com/birdylay/osm-demo.git
    cd osm-demo

Set variables:

    MESH_NAME=osm
    OSM_NAMESPACE=osm-system
    CHART_VERSION=0.9.2

## Install OSM

    helm install $MESH_NAME osm --repo https://openservicemesh.github.io/osm --version $CHART_VERSION --namespace $OSM_NAMESPACE --create-namespace

## Create Namespaces

    kubectl create namespace bookstore
    kubectl create namespace bookbuyer
    kubectl create namespace bookthief
    kubectl create namespace bookwarehouse

## Enable Automatic Sidecar Injection

    kubectl label namespace bookstore openservicemesh.io/monitored-by=$MESH_NAME
    kubectl annotate namespace bookstore openservicemesh.io/sidecar-injection=enabled
    
    kubectl label namespace bookbuyer openservicemesh.io/monitored-by=$MESH_NAME
    kubectl annotate namespace bookbuyer openservicemesh.io/sidecar-injection=enabled
    
    kubectl label namespace bookthief openservicemesh.io/monitored-by=$MESH_NAME
    kubectl annotate namespace bookthief openservicemesh.io/sidecar-injection=enabled
    
    kubectl label namespace bookwarehouse openservicemesh.io/monitored-by=$MESH_NAME
    kubectl annotate namespace bookwarehouse openservicemesh.io/sidecar-injection=enabled

## Deploy Book Demo Microservices

    helm install book-apps book-apps/

## Enable Port Forwarding

    cp .env.example .env
    ./scripts/port-forward-all.sh


## Deploy SMI Access Control Policies

    kubectl apply -f access-manifests/traffic-access-v1.yaml

The counters should now be incrementing for the  `bookbuyer`, and  `bookstore`  applications.  The `bookthief` will not be incrementing:

-   [http://localhost:8080](http://localhost:8080/)  -  **bookbuyer**
-   [http://localhost:8084](http://localhost:8084/)  -  **bookstore**


```kubectl apply -f access-manifests/traffic-access-v1-allow-bookthief.yaml```

The counter in the  `bookthief`  window will start incrementing.

-   [http://localhost:8083](http://localhost:8083/)  -  **bookthief**


## Clean Up

    kubectl delete ns bookbuyer bookthief bookstore bookwarehouse
    helm uninstall book-apps
    helm uninstall osm --namespace $OSM_NAMESPACE
    kubectl delete ns $OSM_NAMESPACE
