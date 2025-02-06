# Deploying MongoDB and Node.js Web Application on Kubernetes using Minikube

This guide will walk you through deploying a MongoDB database and a Node.js web application inside a Kubernetes cluster running on Minikube. The application will interact with MongoDB, and we will deploy everything using Kubernetes YAML files.

## Prerequisites

Before starting, ensure you have the following tools installed:

1. **Docker** – Containerization tool to build and run the Node.js app.  
   - Installation guide: [Docker Installation](https://docs.docker.com/get-docker/)

2. **Minikube** – Tool for running Kubernetes clusters locally.  
   - Installation guide: [Minikube Installation](https://minikube.sigs.k8s.io/docs/)

3. **kubectl** – Command-line tool for interacting with Kubernetes clusters.  
   - Installation guide: [kubectl Installation](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

4. **Node.js** – To develop the Node.js web application.  
   - Installation guide: [Node.js Installation](https://nodejs.org/en/download/)

---

## Step 1: Install Minikube and Start the Cluster

### Start Minikube Cluster
Run the following command to start Minikube with Docker as the VM driver:

```bash
minikube start --vm-driver=docker
```

### Check Minikube Status
Verify if Minikube is running properly:

```bash
minikube status
```

---

## Step 2: Set Up the Node.js Web Application

### Create a Directory for Your Node.js Application
Create a new directory for your application and navigate into it:

```bash
mkdir nodejs-app
cd nodejs-app
```

### Create the Node.js Application Files
Inside the `nodejs-app` directory, create the necessary files for your Node.js application. This includes:
- `app.js` (with basic Express routing and MongoDB interaction)
- `package.json` (with necessary dependencies like express and mongoose)
- `Dockerfile` (to containerize the application)

### Build the Docker Image
Use the following command to build the Docker image for your Node.js application:

```bash
docker build -t nodejs-webapp .
```

### Tag the Docker Image
After building the image, tag it for pushing to a Docker registry:

```bash
docker tag nodejs-webapp yourusername/nodejs-webapp:latest
```

### Push the Docker Image to Docker Hub
Push the Docker image to Docker Hub (or your preferred registry). Ensure you're logged in to Docker:

```bash
docker login
docker push yourusername/nodejs-webapp:latest
```

---

## Step 3: Create Kubernetes Deployment Files

### MongoDB Configuration Files
You will need the following Kubernetes YAML files for MongoDB:
- **`mongo-secret.yaml`**: Contains sensitive information such as MongoDB username and password.
- **`mongo-config.yaml`**: Contains MongoDB configuration settings (like data storage).
- **`mongo.yaml`**: Defines the MongoDB deployment, specifying the container image, environment variables, and volume mounts.

### Node.js Web Application Deployment File
Create a Kubernetes deployment YAML file for the Node.js web application:
- **`webapp.yaml`**: Defines the deployment for your Node.js application, using the Docker image pushed earlier. It also includes the necessary environment variables (such as MongoDB connection details) and any required services or volume mounts.

### Apply Kubernetes Configuration
After creating the YAML files, apply them to your Minikube cluster using the following commands:

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml
```

---

## Step 4: Verify Pods and Services

### Check Pod Status
After applying the YAML files, check the status of the pods:

```bash
kubectl get pods
```

This may take some time, but once the pods are up and running, you can verify the status again to see if they are in the `Running` state.

### Check All Resources
To view all resources like pods, services, deployments, and replica sets, use the following command:

```bash
kubectl get all
```

---

## Step 5: Access the Application

### Port Forwarding
To access the application locally, first find the service port:

```bash
kubectl get svc
```

Then, port forward the service to your local machine:

```bash
kubectl port-forward svc/webapp-service port:port
```

### Access the Web Application
Open your browser and go to:

```
http://127.0.0.1:port
```

You should see the Node.js application running and be able to interact with it.

---

## Step 6: Set Up MongoDB Dashboard

### Deploy MongoDB on kubernetes 
Now, let's create the necessary Kubernetes YAML files for MongoDB, Mongo Express, and the Node.js Web Application.

### 1. MongoDB Secret (`mongo-secret.yaml`)
This file stores sensitive information such as the MongoDB username and password. By using Kubernetes secrets, we ensure that sensitive data is not stored in plain text.

The secret will include `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`, which are required to initialize the MongoDB database.

### 2. MongoDB ConfigMap (`mongo-config.yaml`)
This file defines configuration settings that MongoDB needs for running, such as data directory and port number. It can store non-sensitive configuration data, which can be accessed by MongoDB containers.

You might include parameters like `MONGO_DATA_DIR` for the data storage location or `MONGO_PORT` for specifying the MongoDB service port.

### 3. MongoDB Deployment (`mongo.yaml`)
This file defines the MongoDB deployment. It specifies the following:
- The container image (`mongo:5.0`) to use.
- Replicas: We can specify the number of MongoDB pods to deploy (usually 1 for development or testing).
- Environment Variables: MongoDB will use the secret for credentials (`MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`), and it can reference the config map for the data directory and port.
- Volumes: A persistent volume can be mounted to ensure MongoDB’s data persists across pod restarts.

### 4. Mongo Express Deployment (`mongo-express.yaml`)
Mongo Express is a web-based admin interface for MongoDB. This file defines the deployment for this interface.

- This deployment specifies the image for Mongo Express.
- It will reference the MongoDB service to connect to MongoDB.
- It will also expose a service to make the Mongo Express UI available at a specific port.

### 5. Web Application Deployment (`webapp.yaml`)
This file defines the deployment for your Node.js application.

- The container image (`yourusername/nodejs-webapp:latest`) will be used here (the one built and pushed earlier).
- The deployment will include the MongoDB connection details (e.g., username, password, database name) as environment variables.
- The web app will expose a service to make it accessible externally or internally.

### 6. Service Definition (`service.yaml`)
You will define services for both MongoDB and the Node.js web application.

- The MongoDB service exposes the MongoDB instance internally within the Kubernetes cluster so the web application can access it.
- The Node.js web application service exposes the web application to allow it to handle incoming traffic.

### 7. Ingress (`ingress.yaml`)
This file defines an Ingress resource that allows external HTTP(S) traffic to reach your application.

- This could route traffic from the outside world to the Node.js web application and Mongo Express UI.
- It can include rules for routing traffic to the correct services within the cluster (e.g., `webapp-service` for the Node.js app and `mongo-express-service` for Mongo Express).

### 8. ConfigMap and Secret Integration
- The MongoDB ConfigMap and MongoDB Secret will be injected into the MongoDB container to provide it with configurations and credentials at runtime.
- Similarly, the Node.js web application will use these secrets and ConfigMaps to securely connect to the MongoDB instance.

### Apply Kubernetes Configuration
After creating the above Kubernetes YAML files, apply them to your Minikube cluster using the `kubectl apply` command:

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo.yaml
kubectl apply -f mongo-express.yaml
kubectl apply -f webapp.yaml
kubectl apply -f ingress.yaml
```


### Access MongoDB Dashboard
Once MongoDB Express is deployed, you can access the MongoDB with:

```
minikube service mongo-express-service
```

Log in using the credentials stored in your MongoDB secret.

---

## Step 7: Create MongoDB Database and Collections

### Identify MongoDB Pod Name
Use the following command to identify the MongoDB pod:

```bash
kubectl get pods -l app=mongo
```

### Create a Temporary Pod to Access MongoDB
Run a temporary pod to interact with MongoDB:

```bash
kubectl run mongo-client --rm -it --image=mongo:5.0 -- bash
```

### Connect to MongoDB
Inside the pod, run the following command to connect to MongoDB:

```bash
mongo -u admin -p password --authenticationDatabase admin --host mongodb-service
```

### Create a Database and Insert Data
Once connected to MongoDB, create a new database and insert a sample document:

```bash
use mydb
db.mycollection.insertOne({ name: "John Doe", email: "johndoe@example.com", interests: ["coding", "music"] })
```

### Verify Data
To verify that the data is inserted correctly, run:

```bash
db.mycollection.find()
```

---

## Step 8: Update Node.js Application

### Update Node.js Application
Modify your Node.js application to use the newly created database and collection for data storage.

### Redeploy the Node.js Application
After making updates, redeploy the Node.js application by applying the updated `webapp.yaml` file:

```bash
kubectl apply -f webapp.yaml
```

### Test the Application
Use POST and GET requests from your Node.js application to insert and retrieve data from MongoDB. Verify the changes reflect in MongoDB Express or via the MongoDB CLI.

---

## Conclusion
You have successfully deployed a Node.js web application connected to a MongoDB database on a Kubernetes cluster using Minikube. You can now expand upon this setup by adding additional features, monitoring, and scaling capabilities.

   
