#Dockerization

1. Clone the reporitory git clone
   https://github.com/nyrahul/wisecow

2. Write Dockerfile:
   FROM alpine:latest
   RUN apk add --no-cache bash
   WORKDIR /app
   COPY wisecow.sh /app/wisecow.sh
   CMD ["./wisecow.sh"]

3. Build the Docker Image:
   docker build -t wisecow-app .

4. Run the Docker container to test locally:
   docker run -p 4499:4499 wisecow-app

#Create Kubernetes Manifests:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: your-dockerhub-username/wisecow-app:latest
        ports:
        - containerPort: 4499
        
#Service Menifest:

apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
  - protocol: TCP
    port: 80
    targetPort: 4499
  type: LoadBalancer

#Apply Manifests:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

#Verify Deployment:

kubectl get pods
kubectl get services

#Implement CI/CD Pipeline:

name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and Push Docker Image
      run: |
        docker build -t your-dockerhub-username/wisecow-app:latest .
        docker push your-dockerhub-username/wisecow-app:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'

    - name: Deploy to Kubernetes
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}
      run: |
        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml

#Ingress Manifest:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - localhost
    secretName: wisecow-tls
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-service
            port:
              number: 80

#Apply Ingress Manifest:
kubectl apply -f ingress.yaml

#Verify TLS:
https://localhost
