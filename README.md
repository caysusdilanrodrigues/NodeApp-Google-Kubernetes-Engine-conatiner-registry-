      # Deploy an Application to GKE (Google Kubernetes Engine)
      
      This guide will walk you through deploying a simple Node.js application to Google Kubernetes Engine (GKE) using Docker and Kubernetes.
      
      ## Technologies Used
      
      - Node.js
      - Docker
      - Kubernetes
      - GKE (Google Kubernetes Engine)
      - GCR (Google Container Registry)
      
      ## Steps
      
      ### 1. Create a Kubernetes Cluster on GKE
      
      First, create a Kubernetes cluster on Google Kubernetes Engine through the Google Cloud Console or using the `gcloud` command-line tool.
      
      ### 2. Set Up Connection to the GKE Cluster
      
      Configure `kubectl` to connect to your newly created GKE cluster by running:
      
      ```sh
      gcloud container clusters get-credentials <CLUSTER_NAME> --zone <ZONE> --project <PROJECT_ID>
      
      Replace <CLUSTER_NAME>, <ZONE>, and <PROJECT_ID> with your cluster's name, zone, and project ID.
      
      3. Create a Simple Node.js/Express Application
      Create a simple Node.js application using Express. Ensure you have a basic app.js file and a package.json file for your application.
      
      4. Write a Dockerfile for the Application
      Create a Dockerfile for your application with the following content:
      
      FROM --platform=linux/amd64 node:14
      WORKDIR /usr/app
      COPY package.json .
      RUN npm install
      COPY . .
      EXPOSE 80
      CMD ["node", "app.js"]
      
      
      5. Build the Docker Image
      Build the Docker image using the following command:
      
      docker build -t us.gcr.io/<PROJECT_ID>/imagename\:tag .
      
      Replace <PROJECT_ID>, imagename, and tag with your Google Cloud project ID, desired image name, and tag.
      
      6. Authenticate to GCR
      Authenticate Docker to use Google Container Registry:
      
      gcloud auth configure-docker
      
      7. Push the Docker Image to GCR
      Push the Docker image to Google Container Registry:
      
      docker push us.gcr.io/<PROJECT_ID>/imagename\:tag
      
      
      8. Test the Application Using Docker
      Run the Docker container locally to test your application:
      
      docker run -d -p 3000:80 us.gcr.io/<PROJECT_ID>/imagename\:tag
      
      9. Write Kubernetes Manifest Files
      Deployment Manifest (deploy.yml)
      Create a Kubernetes deployment manifest file:
      
      
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
                image: us.gcr.io/<PROJECT_ID>/imagename\:tag
                ports:
                  - containerPort: 80
      
      
      Service Manifest (service.yml)
      Create a Kubernetes service manifest file:
      
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
      
      
      10. Apply Manifest Files to Create Deployment and Service
      Apply the deployment manifest to create the deployment:
      
      kubectl apply -f deploy.yml
      Check the status of the deployment:
      
      kubectl get deploy
      Apply the service manifest to create the load balancer service:
      
      kubectl apply -f service.yml
      Check the status of the service:
      
      kubectl get svc
      11. Check the External IP of the Service
      Once the service is running, check the external IP of the load balancer in the browser to access your application.
      
      Cleanup
      To clean up and delete the resources created:
      
      kubectl delete -f deploy.yml
      kubectl delete -f service.yml
      Finally, delete your GKE cluster from the Google Cloud Console to avoid incurring unnecessary charges.
      
      This `README.md` file provides a comprehensive guide to deploying a Node.js application to GKE using Docker and Kubernetes. You can customize it further based on your specific needs or additional steps you might have.
      
      
      
      
