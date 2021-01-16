1. **Create K8 cluster**

   ```
   $ gcloud container clusters create mongo-demo \
   --zone asia-southeast1-b \
   --node-locations asia-southeast1-a,asia-southeast1-b,asia-southeast1-c \
   --num-nodes "3"  
   ```

   **Create a cluster role binding for the user**

   ```
   $ kubectl create clusterrolebinding cluster-admin-binding \
   --clusterrole cluster-admin \
   --user $(gcloud config get-value account)
   ```

   **Resize cluster**

   ```
   $ gcloud constainer clusters resize mongo-demo \
   --node-pool default-pool \
   --num-nodes <nodeNumber>
   ```
   
   Hi there
