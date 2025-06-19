# Deploy an Application to GKE (Google Kubernetes Engine)

This guide will walk you through deploying a simple Node.js application to Google Kubernetes Engine (GKE) using Docker and Kubernetes.

##  Technologies Used

* Node.js
* Docker
* Kubernetes
* GKE (Google Kubernetes Engine)
* GCR (Google Container Registry)

---

## Steps

### 1. Create a Kubernetes Cluster on GKE

Create a Kubernetes cluster via the Google Cloud Console or with the CLI:

```bash
gcloud container clusters create <CLUSTER_NAME> \
  --region <REGION> \
  --enable-ip-alias \
  --enable-private-nodes \
  --project <PROJECT_ID>
```

---

### 2. Set Up Connection to the GKE Cluster

Configure `kubectl`:

```bash
gcloud container clusters get-credentials <CLUSTER_NAME> --region <REGION> --project <PROJECT_ID>
```

Replace `<CLUSTER_NAME>`, `<REGION>`, and `<PROJECT_ID>` accordingly.

---

### 3. Create a Simple Node.js/Express Application

Create a basic `app.js` and `package.json` for your Node.js app. Example:

**app.js**

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Node.js on GKE!');
});

app.listen(80, () => {
  console.log('Server is running on port 80');
});
```

**package.json**

```json
{
  "name": "nodeapp",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

---

### 4. Write a Dockerfile

**Dockerfile**

```Dockerfile
FROM --platform=linux/amd64 node:14
WORKDIR /usr/app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "app.js"]
```

---

### 5. Build the Docker Image

```bash
docker build -t us.gcr.io/<PROJECT_ID>/nodeapp:v1 .
```

---

### 6. Authenticate to GCR

```bash
gcloud auth configure-docker
```

---

### 7. Push the Docker Image to GCR

```bash
docker push us.gcr.io/<PROJECT_ID>/nodeapp:v1
```

---

### 8. Test the Application Locally

```bash
docker run -d -p 3000:80 us.gcr.io/<PROJECT_ID>/nodeapp:v1
```

Open `http://localhost:3000` to verify.

---

### 9. Write Kubernetes Manifest Files

**deploy.yml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodeappdeployment
  labels:
    type: backend
    app: nodeapp
spec:
  replicas: 1
  selector:
    matchLabels:
      type: backend
      app: nodeapp
  template:
    metadata:
      name: nodeapppod
      labels:
        type: backend
        app: nodeapp
    spec:
      containers:
        - name: nodecontainer
          image: us.gcr.io/<PROJECT_ID>/nodeapp:v1
          ports:
            - containerPort: 80
```

**service.yml**

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nodeapp-load-service
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    type: backend
    app: nodeapp
  type: LoadBalancer
```

---

### 10. Apply the Manifests

```bash
kubectl apply -f deploy.yml
kubectl get deploy
kubectl apply -f service.yml
kubectl get svc
```

---

### 11. Access the Application

After a few minutes, your `nodeapp-load-service` will have an **EXTERNAL-IP**. Access the app in a browser:

```text
http://<EXTERNAL-IP>
```

---

##  Cleanup

To avoid charges, delete resources:

```bash
kubectl delete -f deploy.yml
kubectl delete -f service.yml
gcloud container clusters delete <CLUSTER_NAME> --region <REGION>
```

---

## ✅ Summary

This guide helps you containerize a Node.js app and deploy it to GKE using Docker, GCR, and Kubernetes. Customize as needed!
