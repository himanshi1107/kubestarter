# KIND Cluster Setup Guide

## 1. Installing Docker, KIND and kubectl
Install KIND and kubectl using the provided script:
```bash

#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

echo "Checking if Docker is already installed..."
if ! command -v docker &> /dev/null; then
    echo "Installing Docker..."
    # Install Docker using the convenience script
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    rm get-docker.sh

    # Start Docker and enable it on boot
    sudo systemctl start docker
    sudo systemctl enable docker

    # Add the current user to the docker group
    sudo usermod -aG docker $USER
    echo "Docker installed successfully."
else
    echo "Docker is already installed. Skipping installation."
fi

echo "Installing Kind..."
# Download Kind binary
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

# Make Kind executable and move it to /usr/local/bin
chmod +x kind
sudo mv kind /usr/local/bin/

# Verify Kind installation
kind version
echo "Kind installed successfully."

echo "Installing kubectl..."
# Use a fixed version for kubectl
KUBECTL_VERSION="v1.28.0"  # Replace with your desired Kubernetes version
curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"

# Make kubectl executable and move it to /usr/local/bin
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify kubectl installation
kubectl version --client
echo "kubectl installed successfully."

echo "Setup complete! Please log out and log back in for Docker permissions to take effect."

```

## 2. Setting Up the KIND Cluster
Create a config.yaml file:

```yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443   
    protocol: TCP 
```
Create the cluster using the configuration file:

```bash

 kind create cluster --name tws-cluster --config config.yml
```
Verify the cluster:

```bash

kubectl get nodes
kubectl cluster-info
```
## 3. Accessing the Cluster
Use kubectl to interact with the cluster:
```bash

kubectl cluster-info --context kind-tws-cluster
```

## 4. Setting Up the Kubernetes Dashboard
Deploy the Dashboard
Apply the Kubernetes Dashboard manifest:
```bash

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
Create an Admin User
Create a dashboard-admin-user.yml file with the following content:

```yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
Apply the configuration:

```bash

kubectl apply -f dashboard-admin-user.yml
```
Get the Access Token
Retrieve the token for the admin-user:

```bash

kubectl -n kubernetes-dashboard create token admin-user
```
Copy the token for use in the Dashboard login.

Access the Dashboard
Start the Dashboard using kubectl proxy:

```bash

kubectl proxy
```
Open the Dashboard in your browser:

```bash

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
Use the token from the previous step to log in.

## 5. Deleting the Cluster
Delete the KIND cluster:
```bash

kind delete cluster --name my-kind-cluster
```

## 6. Notes

Multiple Clusters: KIND supports multiple clusters. Use unique --name for each cluster.
Custom Node Images: Specify Kubernetes versions by updating the image in the configuration file.
Ephemeral Clusters: KIND clusters are temporary and will be lost if Docker is restarted.
