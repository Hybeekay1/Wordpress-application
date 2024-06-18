*Prerequisites*

- Kubernetes cluster (e.g., Minikube, GKE, AKS)
- Git repository
- Bitnami Sealed Secrets installed on your machine
- WordPress Docker image
- Kubernetes CLI (kubectl)

*Creating Encrypted Secrets*

1. Create a YAML file (e.g., `secrets.yaml`) with your database credentials:
```
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-db-credentials
type: (link unavailable)
stringData:
  username: <your-username>
  password: <your-password>
```
Replace `<your-username>` and `<your-password>` with your actual database credentials.

1. Encrypt the secrets using Bitnami Sealed Secrets:
```
kubeseal --format yaml < secrets.yaml > sealed-secrets.yaml
```
This will generate an encrypted `sealed-secrets.yaml` file.

*Storing Secrets in Git*

1. Add the encrypted `sealed-secrets.yaml` file to your Git repository:
```
git add sealed-secrets.yaml
git commit -m "Add encrypted secrets"
git push origin main
```
*Creating a WordPress Deployment*

1. Create a YAML file (e.g., `deployment.yaml`) for your WordPress deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        env:
        - name: WORDPRESS_DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: wordpress-db-credentials
              key: username
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wordpress-db-credentials
              key: password
```
This deployment uses environment variables for database credentials, which will be populated from the encrypted secrets.

*Configuring Environment Variables*

1. Create a YAML file (e.g., `env.yaml`) with environment variable definitions:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-env
data:
  WORDPRESS_DB_HOST: <your-db-host>
  WORDPRESS_DB_NAME: <your-db-name>
```
Replace `<your-db-host>` and `<your-db-name>` with your actual database host and name.

*Applying the Deployment*

1. Apply the deployment YAML files:
```
bash
kubectl apply -f deployment.yaml env.yaml
```
*Verifying the Deployment*

1. Check the deployment status:
```
kubectl get deployments
```
1. Verify that the WordPress container is running:
```
kubectl get pods
```
1. Access your WordPress site using the service name or IP address.

Congratulations! You have successfully deployed WordPress using encrypted secrets stored in Git and managed by Kubernetes.
