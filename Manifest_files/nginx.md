### Manifest File Examples:

1. **Nginx Pod Manifest in "nginx" Namespace:**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-pod
     namespace: nginx
   spec:
     containers:
       - name: nginx-container
         image: nginx:latest
         ports:
         - containerPort: 80 
   ```

2. **Nginx Service Manifest in "nginx" Namespace:**

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
     namespace: nginx
   spec:
     selector:
       app: nginx-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
   ```

3. **Nginx Deployment Manifest in "nginx" Namespace:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
     namespace: nginx
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx-app
     template:
       metadata:
         labels:
           app: nginx-app
       spec:
         containers:
           - name: nginx-container
             image: nginx:latest
   ```

### Nginx Namespace Deployment Steps:

1. **Create the "nginx" Namespace:**

   ```sh
   kubectl create namespace nginx
   ```

2. **Apply the Nginx Pod, Service, and Deployment YAMLs within the "nginx" Namespace:**

   ```sh
   kubectl apply -f nginx-pod.yaml -n nginx
   kubectl apply -f nginx-service.yaml -n nginx
   kubectl apply -f nginx-deployment.yaml -n nginx
   ```
