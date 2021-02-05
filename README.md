## Overview
We will deploy a replicated database, configure its persistence and make it available with a UI client from browser using Ingress. We will also use Helm to make the process more efficient.

In detail we will deploy MongoDB on Google Kubernetes Engine using Helm. We will create replicated MongoDB using StatefulSet component and configure data persistence for MongoDB with Google Cloud Platform's cloud storage. Then we will deploy a MongoExpress, a UI client, for MongoDB database to access it from the browser. For this client we will configure NGINX Ingress Controller. So we will deploy Ingress Controller in the cluster and configure Ingress rule to route the request to MongoExpress internal Service.

### Installation
1. **Create K8 cluster**

   ```
   $ gcloud container clusters create mongodb-gke-demo \
   --zone asia-southeast1-b \
   --num-nodes "3"  
   ```

2. **Add Bitnami helm charts**

   ```
   $ helm repo add bitnami https://charts.bitnami.com/bitnami
   $ helm repo update
   ```

3. **Configure data persistence for MongoDB with GKE's SSD storage**

   ```
   $ kubectl apply -f mongodb-storageclass.yaml 
   ```
   This is an optional step: if you choose to skip this, comment out lines 3-4 in the mongodb-values.yaml file 
   ```
   $ nano mongodb-values.yaml
   
   #persistence:
   #  storageClass: ssd-storageclass
   ```
   This will configure data persistence instead using standard persistent disks based on the default GKE storage class

4. **Deploy replicated MongoDB using StatefulSet component**

   ```
   $ helm install mongodb --values mongodb-values.yaml bitnami/mongodb
   ```

5. **Deploy MongoExpress UI client for MongoDB database to be accessed from the browser**
   
   ```
   $ kubectl apply -f mongo-express.yaml
   ```

6. **Deploy NGINX Ingress Controller**

   ```
   $ helm repo add add nginx-stable https://helm.nginx.com/stable
   $ helm repo update
   $ helm install nginx-ingress nginx-stable/nginx-ingress
   ```

7. **Configure Ingress rule to route traffic to MongoExpress internal service**
   
   GKE won't create a domain name. However, you can configure the load balancer to use a static IP address that you have reserved. To reserve a global static IP address ie. mongo-demo-ip
   ```
   $ gcloud compute addresses create mongo-demo-ip --global
   ```
   To view the details of the reservation
   ```
   $ gcloud compute addresses describe mongo-demo-ip --global
   ```
   To configure ingress rule
   ```
   $ kubectl apply -f ingress.yaml
   ```
   To verify the ingress
   ```
   $ kubectl get ingress
   ```
   View the MongoExpress UI client from the browser
    ```
   http://<ADDRESS>
   ```
   
### Clean up
```
$ kubectl delete -f ingress.yaml
$ gcloud compute addresses delete mongo-demo-ip --global
$ helm uninstall nginx-ingress
$ kubectl delete -f mongo-express.yaml
$ helm uninstall mongodb
$ kubectl delete -f mongodb-storageclass.yaml
$ gcloud container clusters delete mongodb-gke-demo
$ gcloud compute disks list
$ gcloud compute disks delete <diskName>
```
**Remember to delete the actual, physical storage in Google Compute Engine persistent disks**

### Prometheus monitoring
```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

$ helm install prometheus prometheus-community/kube-prometheus-stack

$ helm show values prometheus-community/prometheus-mongodb-exporter > values.yaml
$ helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f values.yaml

$ kubectl get service
$ kubectl port-forward service mongodb-exporter-prometheus-mongodb-exporter <portNum>
$ kubectl get deployment
$ kubectl port-forward deployment/prometheus-grafana <portNum> 
```
