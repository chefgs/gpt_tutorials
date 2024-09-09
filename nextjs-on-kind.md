# Deploy Nextjs App in Kind Cluster

To deploy a simple Next.js UI app in a `kind` (Kubernetes in Docker) cluster, you’ll follow a step-by-step process that includes setting up the `kind` cluster, containerizing the Next.js app, deploying it on Kubernetes using the cluster, and finally exposing the service.

Here’s the detailed process:

### Step 1: Install Required Tools
Before you begin, you need to install the following tools:
- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Node.js](https://nodejs.org/) (for building the Next.js app)
  
Make sure Docker is running on your system.

### Step 2: Create a Simple Next.js App

1. First, create a basic Next.js app:
   ```bash
   npx create-next-app@latest nextjs-kind-demo
   cd nextjs-kind-demo
   ```

2. Start the Next.js development server to ensure everything is working:
   ```bash
   npm run dev
   ```

   You should be able to access the app on `http://localhost:3000`.

### Step 3: Create a Dockerfile for the Next.js App

1. Create a `Dockerfile` in the root of your Next.js project:
   ```Dockerfile
   # Stage 1: Build the application
   FROM node:18-alpine AS builder
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   RUN npm run build

   # Stage 2: Serve the application
   FROM node:18-alpine
   WORKDIR /app
   COPY --from=builder /app ./
   EXPOSE 3000
   CMD ["npm", "run", "start"]
   ```

   This `Dockerfile` has two stages: one for building the app and the other for serving the built app.

### Step 4: Build the Docker Image

1. Build the Docker image using the Dockerfile:
   ```bash
   docker build -t nextjs-kind-demo .
   ```

2. Verify the image was built successfully:
   ```bash
   docker images
   ```

### Step 5: Set Up a `kind` Kubernetes Cluster

1. Create a `kind` cluster with the following command:
   ```bash
   kind create cluster --name nextjs-cluster
   ```

2. Check that the cluster is running:
   ```bash
   kubectl cluster-info --context kind-nextjs-cluster
   ```

### Step 6: Load the Docker Image into the `kind` Cluster

Since `kind` runs Kubernetes in Docker, you need to load the Docker image into the cluster:
```bash
kind load docker-image nextjs-kind-demo --name nextjs-cluster
```

### Step 7: Create Kubernetes Deployment and Service YAML Files

1. Create a deployment YAML file named `deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nextjs-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: nextjs
     template:
       metadata:
         labels:
           app: nextjs
       spec:
         containers:
         - name: nextjs
           image: nextjs-kind-demo:latest
           ports:
           - containerPort: 3000
   ```

2. Create a service YAML file named `service.yaml`:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nextjs-service
   spec:
     selector:
       app: nextjs
     ports:
     - protocol: TCP
       port: 80
       targetPort: 3000
     type: LoadBalancer
   ```

### Step 8: Deploy to the `kind` Cluster

1. Apply the deployment:
   ```bash
   kubectl apply -f deployment.yaml
   ```

2. Apply the service:
   ```bash
   kubectl apply -f service.yaml
   ```

3. Check the status of the pods:
   ```bash
   kubectl get pods
   ```

4. Check the services to see if the app is accessible:
   ```bash
   kubectl get services
   ```

### Step 9: Access the Next.js App in the Kind Cluster

`kind` does not have a built-in load balancer, but you can use port-forwarding to access the app from your local machine.

1. Use `kubectl` to forward the port:
   ```bash
   kubectl port-forward service/nextjs-service 8080:80
   ```

2. Access the Next.js app in your browser at `http://localhost:8080`.

### Step 10: Cleanup

When you are done, you can delete the `kind` cluster:
```bash
kind delete cluster --name nextjs-cluster
```

### Directory Structure
Here is a sample directory structure after you complete all the steps:

```
nextjs-kind-demo/
├── public/
├── pages/
├── styles/
├── node_modules/
├── Dockerfile
├── package.json
├── deployment.yaml
├── service.yaml
└── .dockerignore
```

### Summary of Steps

1. Create a Next.js app.
2. Containerize the app using a `Dockerfile`.
3. Set up a `kind` Kubernetes cluster.
4. Load the Docker image into the `kind` cluster.
5. Deploy the app to Kubernetes using deployment and service YAMLs.
6. Access the app using port-forwarding.

### Next Suggested Prompts:

- "How can I add persistence to a Kubernetes service in kind cluster?"
- "How can I configure a CI/CD pipeline for deploying updates to the Next.js app in Kubernetes?"
- "What is the best way to manage secrets in a Kubernetes deployment?"