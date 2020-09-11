#### Task 1: Download a sample app from GitHub

1. Run in cloud shell
```
mkdir gcp-course
```
2. Navigate into newly created directory
```
cd gcp-course
```
3. Clone a simple Python Flask app from GitHub:
```
git clone https://GitHub.com/GoogleCloudPlatform/training-data-analyst.git
```
4. Change to the deploying-apps-to-gcp folder:
```
cd training-data-analyst/courses/design-process/deploying-apps-to-gcp

```
5. To test the program, enter the following command to install requirements and then start the program:
```
sudo pip3 install -r requirements.txt
python3 main.py
```

#### Task 2: Deploy to App Engine
1. Create yaml file
```
cat 'runtime: python37' > app.yaml
```
2. Create App Engine App
```
gcloud app create --region=us-central
```
3. Deploy app
```
gcloud app deploy --version=one --quiet
```
4. To view app run
```
curl https://$DEVSHELL_PROJECT_ID.appspot.com
```
5. Change App title
```
sed -i -e 's/GCP/App Engine/g' main.py
```
6. Deploy a version two of the app
```
gcloud app deploy --version=two --no-promote --quiet
```

#### Task 3: Deploy to Kubernetes Engine
1. Create a cluster
```
gcloud beta container --project "qwiklabs-gcp-03-054e3d70a7fc" clusters create "cluster-2" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.15.12-gke.2" --machine-type "e2-medium" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-stackdriver-kubernetes --enable-ip-alias --network "projects/qwiklabs-gcp-03-054e3d70a7fc/global/networks/default" --subnetwork "projects/qwiklabs-gcp-03-054e3d70a7fc/regions/us-central1/subnetworks/default" --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0
```
2. Connect to the cluster
```
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project qwiklabs-gcp-03-054e3d70a7fc
```
3. To test your connection, enter the following command:
```
kubectl get nodes
```
4. Change App title
```
sed -i -e 's/App Engine/Kubernetes Engine/g' main.py
```
5. Create kubernetes-config.yaml
```
cat '---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: devops-deployment
  labels:
    app: devops
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops
      tier: frontend
  template:
    metadata:
      labels:
        app: devops
        tier: frontend
    spec:
      containers:
      - name: devops-demo
        image: <YOUR IMAGE PATH HERE>
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: devops-deployment-lb
  labels:
    app: devops
    tier: frontend-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: devops
    tier: frontend' > kubernetes-config.yaml
```
6. Build a Docker image
```
cd ~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp
gcloud builds submit --tag gcr.io/$DEVSHELL_PROJECT_ID/devops-image:v0.2 .
```
7. Replace image path in yaml
```
sed -i -e 's/<YOUR IMAGE PATH HERE>/gcr.io/$DEVSHELL_PROJECT_ID/devops-image:v0.2/g' kubernetes-config.yaml
```
8. Deploy the application
```
kubectl apply -f kubernetes-config.yaml
```
9. View instances
```
kubectl get pods
```
10. View services
```
kubectl get services
```
11. Test application
```
curl <EXTERNAL_IP>
```


#### Task 4: Deploy to Cloud Run
1. Change App title
```
sed -i -e 's/Kubernetes Engine/Cloud Run/g' main.py
```
2. Create a Docker image for Cloud Run
```
cd ~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp
gcloud builds submit --tag gcr.io/$DEVSHELL_PROJECT_ID/cloud-run-image:v0.1 .
```
3. Create a Cloud Run service
```
gcloud run deploy my-service \
--image=gcr.io/qwiklabs-gcp-03-054e3d70a7fc/cloud-run-image@sha256:006056c70364f0607bf5aae6c93ca576dbf31feac33b2131b774853cf5f38a7b \
--allow-unauthenticated \
--platform=managed \
--region=us-central1 \
--project=qwiklabs-gcp-03-054e3d70a7fc
```
