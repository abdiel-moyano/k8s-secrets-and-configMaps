
# Configuration Management with ConfigMaps and Secrets in Kubernetes

This lab aims to demonstrate how to manage configurations and sensitive data in Kubernetes using **ConfigMaps** and **Secrets**. You will learn to pass environment variables to containers and integrate these resources into a sample application.

## Lab Objectives

- Create and manage a ConfigMap for non-sensitive configurations.
- Create and manage a Secret for sensitive data.
- Use the ConfigMap and Secret in a sample application as environment variables.

## Prerequisites

- Access to a Kubernetes cluster (Minikube for local testing or a managed cloud cluster).
- `kubectl` installed and configured to connect to your Kubernetes cluster.

## Lab Steps

### 1. Create the ConfigMap

1. Define a ConfigMap YAML file (`example-configmap.yaml`) with configuration data:
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: example-configmap
   data:
     APP_NAME: "MyApplication"
     APP_ENV: "production"
   ```

2. Apply the ConfigMap to the cluster:
   ```bash
   kubectl apply -f example-configmap.yaml
   ```

### 2. Create the Secret

1. Define a Secret YAML file (`example-secret.yaml`) with sensitive data:
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: example-secret
   type: Opaque
   data:
     USER_NAME: c2VjcmV0VXNlcg==  # Base64-encoded (e.g., "secretUser")
     PASSWORD: c2VjcmV0UGFzc3dvcmQ=  # Base64-encoded (e.g., "secretPassword")
   ```

2. Apply the Secret to the cluster:
   ```bash
   kubectl apply -f example-secret.yaml
   ```

### 3. Create a Deployment to Use ConfigMap and Secret

1. Define a Deployment YAML file (`example-deployment.yaml`) that uses ConfigMap and Secret as environment variables:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: example-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: example-app
     template:
       metadata:
         labels:
           app: example-app
       spec:
         containers:
           - name: example-container
             image: nginx
             env:
               - name: APP_NAME
                 valueFrom:
                   configMapKeyRef:
                     name: example-configmap
                     key: APP_NAME
               - name: APP_ENV
                 valueFrom:
                   configMapKeyRef:
                     name: example-configmap
                     key: APP_ENV
               - name: USER_NAME
                 valueFrom:
                   secretKeyRef:
                     name: example-secret
                     key: USER_NAME
               - name: PASSWORD
                 valueFrom:
                   secretKeyRef:
                     name: example-secret
                     key: PASSWORD
   ```

2. Deploy the application:
   ```bash
   kubectl apply -f example-deployment.yaml
   ```

### 4. Verify the Environment Variables

1. Get the name of the running pod:
   ```bash
   kubectl get pods -l app=example-app
   ```

2. Access the pod's shell to confirm the environment variables:
   ```bash
   kubectl exec -it <pod_name> -- /bin/sh
   ```

3. Inside the container, check the environment variables:
   ```sh
   echo $APP_NAME
   echo $APP_ENV
   echo $USER_NAME
   echo $PASSWORD
   ```

### 5. Clean Up

Remove the resources created in the lab:
```bash
kubectl delete -f example-configmap.yaml
kubectl delete -f example-secret.yaml
kubectl delete -f example-deployment.yaml
```

## Conclusion

This lab provides an overview of how to use ConfigMaps and Secrets in Kubernetes to manage application configuration and sensitive data securely.
