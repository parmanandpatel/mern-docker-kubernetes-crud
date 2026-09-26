# MERN CRUD — Docker, Docker Hub & Kubernetes

This README is a step-by-step reference for taking a MERN CRUD application from local development to Docker, Docker Hub, and Kubernetes.

---



## 1. Technology Stack

* React.js — Frontend
* Node.js + Express.js — Backend
* MongoDB — Database
* Docker — Containerization
* Docker Hub — Docker image registry
* Kubernetes — Container orchestration

### Overall Flow

```text
MERN Application
      ↓
Local Testing
      ↓
Docker
      ↓
Docker Compose
      ↓
Docker Hub
      ↓
Kubernetes
      ↓
Ingress
      ↓
Production
```

---

# PART 1 — Install Docker

## 2. Install Docker Desktop

Install Docker Desktop on Windows.

After installation, verify:

```powershell
docker --version
```

Check Docker:

```powershell
docker info
```

Check Docker Compose:

```powershell
docker compose version
```

Docker must be running before continuing.

---

# PART 2 — Create Docker Hub Account

## 3. Create Docker Hub Account

Create an account on Docker Hub.

Example username:

```text
parampatel123456
```

Login:

```powershell
docker login
```

Enter your Docker Hub credentials.

---

# PART 3 — Verify MERN Application Locally

## 4. First Make Sure the Application Works Without Docker

Before Dockerizing anything, verify the application normally.

### Backend

```powershell
cd express-mongodb-crud
npm install
npm start
```

Backend:

```text
http://localhost:5000
```

Test API:

```text
http://localhost:5000/api/products
```

### Frontend

```powershell
cd frontend
npm install
npm run dev
```

Frontend:

```text
http://localhost:5173
```

Verify:

```text
[ ] Frontend opens
[ ] Backend starts
[ ] MongoDB connects
[ ] GET works
[ ] POST works
[ ] PUT works
[ ] DELETE works
```

**Do not move to Docker until the normal application works.**

---

# PART 4 — Create Docker Images

## 5. Backend Dockerfile

Create:

```text
express-mongodb-crud/Dockerfile
```

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

Build:

```powershell
docker build -t parampatel123456/product-crud-backend:latest ./express-mongodb-crud
```

Verify:

```powershell
docker images
```

---

## 6. Frontend Dockerfile

Create:

```text
frontend/Dockerfile
```

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

Build:

```powershell
docker build -t parampatel123456/product-crud-frontend:latest ./frontend
```

Verify:

```powershell
docker images
```

---

# PART 5 — Verify Everything Using Docker Compose

## 7. Create docker-compose.yml

```yaml
services:
  app:
    build:
      context: ./express-mongodb-crud
    container_name: express-crud-api
    ports:
      - "5000:5000"
    environment:
      PORT: 5000
      MONGO_URI: mongodb://mongo:27017/productdb
    depends_on:
      - mongo

  frontend:
    build:
      context: ./frontend
    container_name: react-frontend
    ports:
      - "3000:5173"
    depends_on:
      - app

  mongo:
    image: mongo:8
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

Start:

```powershell
docker compose up -d --build
```

Check:

```powershell
docker ps
```

Expected:

```text
react-frontend
express-crud-api
mongodb
```

Test:

```text
Frontend:
http://localhost:3000

Backend:
http://localhost:5000/api/products
```

### Important

Follow this rule:

```text
Local Application
      ↓
Docker Image
      ↓
Docker Compose
      ↓
Verify Everything
      ↓
Docker Hub
      ↓
Kubernetes
```

---

# PART 6 — Push Images to Docker Hub

## 8. Login

```powershell
docker login
```

## 9. Push Backend

```powershell
docker push parampatel123456/product-crud-backend:latest
```

## 10. Push Frontend

```powershell
docker push parampatel123456/product-crud-frontend:latest
```

Docker Hub should now contain:

```text
parampatel123456/product-crud-backend
parampatel123456/product-crud-frontend
```

---

# PART 7 — Install Kubernetes

## 11. Enable Kubernetes

For local learning, Docker Desktop Kubernetes can be used.

Enable Kubernetes from:

```text
Docker Desktop
    ↓
Settings
    ↓
Kubernetes
    ↓
Enable Kubernetes
```

Wait until Kubernetes is running.

---

## 12. Verify Kubernetes

Check kubectl:

```powershell
kubectl version --client
```

Check cluster:

```powershell
kubectl cluster-info
```

Check nodes:

```powershell
kubectl get nodes
```

Expected:

```text
NAME                    STATUS   ROLES
desktop-control-plane   Ready    control-plane
```

The important value is:

```text
STATUS = Ready
```

---

# PART 8 — Create Kubernetes Folder

## 13. Project Structure

```text
project/
│
├── express-mongodb-crud/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── docker-compose.yml
│
└── k8s/
    ├── mongo-deployment.yml
    ├── mongo-service.yml
    ├── backend-deployment.yml
    ├── backend-service.yml
    ├── frontend-deployment.yml
    └── frontend-service.yml
```

---

# PART 9 — MongoDB Kubernetes Deployment

## 14. mongo-deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:8
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-data
              mountPath: /data/db
      volumes:
        - name: mongo-data
          emptyDir: {}
```

> `emptyDir` is okay for local learning. For production, use PersistentVolume/PersistentVolumeClaim.

---

# PART 10 — MongoDB Service

## 15. mongo-service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
  type: ClusterIP
```

Backend connects to MongoDB using:

```text
mongodb://mongo-service:27017/productdb
```

Important:

```text
Docker Compose:
mongodb://mongo:27017/productdb

Kubernetes:
mongodb://mongo-service:27017/productdb
```

---

# PART 11 — Backend Kubernetes Deployment

## 16. backend-deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: parampatel123456/product-crud-backend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 5000
          env:
            - name: PORT
              value: "5000"
            - name: MONGO_URI
              value: "mongodb://mongo-service:27017/productdb"
```

---

# PART 12 — Backend Kubernetes Service

## 17. backend-service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: ClusterIP
```

Backend is now available internally as:

```text
backend-service:5000
```

---

# PART 13 — Frontend Kubernetes Deployment

## 18. frontend-deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: parampatel123456/product-crud-frontend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 5173
```

---

# PART 14 — Frontend Kubernetes Service

## 19. frontend-service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 5173
  type: NodePort
```

---

# PART 15 — Deploy to Kubernetes

## 20. Apply All YAML Files

Go into the Kubernetes folder:

```powershell
cd k8s
```

Apply:

```powershell
kubectl apply -f .
```

Check Pods:

```powershell
kubectl get pods
```

Expected:

```text
backend-xxxxx    1/1   Running
frontend-xxxxx   1/1   Running
mongo-xxxxx      1/1   Running
```

Check Services:

```powershell
kubectl get services
```

---

# PART 16 — Access Frontend

## 21. Find NodePort

```powershell
kubectl get services
```

Example:

```text
frontend-service   NodePort   ...   3000:31150/TCP
```

Then:

```text
http://localhost:31150
```

For Docker Desktop/kind, port-forwarding is also reliable:

```powershell
kubectl port-forward service/frontend-service 3000:3000
```

Then:

```text
http://localhost:3000
```

---

# PART 17 — Test Backend

## 22. Port Forward Backend

Backend is a `ClusterIP`, so expose it temporarily:

```powershell
kubectl port-forward service/backend-service 5000:5000
```

Test:

```text
http://localhost:5000/api/products
```

---

# PART 18 — Kubernetes Troubleshooting

## 23. Check Pods

```powershell
kubectl get pods
```

## 24. Check Deployments

```powershell
kubectl get deployments
```

## 25. Check Services

```powershell
kubectl get services
```

## 26. Check Everything

```powershell
kubectl get all
```

## 27. Backend Logs

```powershell
kubectl logs deployment/backend
```

## 28. Frontend Logs

```powershell
kubectl logs deployment/frontend
```

## 29. MongoDB Logs

```powershell
kubectl logs deployment/mongo
```

## 30. Detailed Pod Information

```powershell
kubectl describe pod <pod-name>
```

---

# PART 19 — Important Concepts

| Concept          | Meaning                                         |
| ---------------- | ----------------------------------------------- |
| Docker           | Runs applications in containers                 |
| Image            | Package containing application and dependencies |
| Docker Hub       | Stores Docker images                            |
| Kubernetes       | Manages containers                              |
| Pod              | Smallest Kubernetes workload                    |
| Deployment       | Manages Pods                                    |
| Service          | Provides stable networking                      |
| ClusterIP        | Internal service                                |
| NodePort         | Exposes service through a port                  |
| Replicas         | Number of Pod instances                         |
| ConfigMap        | Stores non-sensitive configuration              |
| Secret           | Stores sensitive configuration                  |
| Ingress          | Routes HTTP/HTTPS traffic                       |
| PersistentVolume | Persistent storage                              |
| Readiness Probe  | Determines whether Pod can receive traffic      |
| Liveness Probe   | Determines whether container is healthy         |

---

# PART 20 — Next Kubernetes Learning Path

After this basic deployment, learn these topics in order:

```text
1. Ingress
      ↓
2. Ingress Controller / Nginx
      ↓
3. ConfigMap
      ↓
4. Kubernetes Secrets
      ↓
5. PersistentVolume + PVC
      ↓
6. Readiness Probe
      ↓
7. Liveness Probe
      ↓
8. Scaling / Replicas
      ↓
9. Rolling Updates
      ↓
10. Resource Requests & Limits
      ↓
11. Namespaces
      ↓
12. Horizontal Pod Autoscaler
      ↓
13. Kubernetes Networking
      ↓
14. AWS EKS
      ↓
15. GitHub Actions CI/CD
      ↓
16. Production Kubernetes Deployment
```

---

# Quick Checklist

Use this checklist for future projects:

```text
[ ] Install Docker Desktop
[ ] Verify Docker
[ ] Create Docker Hub account
[ ] docker login
[ ] Verify MERN application locally
[ ] Create backend Dockerfile
[ ] Create frontend Dockerfile
[ ] Build Docker images
[ ] Test Docker images
[ ] Create docker-compose.yml
[ ] Test complete application with Docker Compose
[ ] Push images to Docker Hub
[ ] Install/enable Kubernetes
[ ] Verify kubectl
[ ] Verify Kubernetes node
[ ] Create k8s folder
[ ] Create MongoDB Deployment
[ ] Create MongoDB Service
[ ] Create Backend Deployment
[ ] Create Backend Service
[ ] Create Frontend Deployment
[ ] Create Frontend Service
[ ] kubectl apply -f .
[ ] Check Pods
[ ] Check Services
[ ] Test Frontend
[ ] Test Backend
[ ] Check logs if required
[ ] Learn Ingress
```

# Golden Rule

Always remember:

```text
CODE
  ↓
LOCAL TESTING
  ↓
DOCKERFILE
  ↓
DOCKER IMAGE
  ↓
DOCKER COMPOSE
  ↓
TEST EVERYTHING
  ↓
DOCKER HUB
  ↓
KUBERNETES
  ↓
DEPLOYMENTS
  ↓
SERVICES
  ↓
INGRESS
  ↓
PRODUCTION
```

This sequence gives you a repeatable workflow for deploying MERN applications using Docker and Kubernetes.
