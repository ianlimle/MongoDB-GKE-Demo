1. **Create K8 cluster**

   ```
   $ gcloud container clusters create mongo-demo \
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
   This is optional step: if you choose to skip this, comment out 'persistence' values in the mongodb-values.yaml file ie. 
   ```
   architecture: replicaset
   replicaCount: 3
   #persistence:
   #  storageClass: ssd-storageclass
   auth:
     password: secret-root-pwd
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
   $ 
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
