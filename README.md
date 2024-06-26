# task: Create a wordpress deployment, which uses kubernetes secrets for it's database credentials environment variables. The kubernetes secrets should be stored in Git and must be encrypted using Bitnami Sealed Secrets

*Prerequisites*

- Kubernetes cluster (e.g., Minikube, GKE, AKS)
- Git repository
- Bitnami Sealed Secrets CLI and the controller
- WordPress Docker image
- Kubernetes CLI (kubectl)

*Creating Encrypted Secrets*

1. Create a YAML file (e.g., `secrets.yaml`) with your database credentials:
```
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-db-credentials
type: 
stringData:
  username: <your-username>
  password: <your-password>
```
Replace `<your-username>` and `<your-password>` with your actual database credentials.

check if the sealed-secret cli is installed
```
kubeseal --fetch-cert --controller-name my-sealed-secrets --controller-namespace kube-system

```

1. Encrypt the secrets using Bitnami Sealed Secrets:
```
kubeseal --fetch-cert --controller-name my-sealed-secrets --controller-namespace kube-system

kubeseal --controller-name my-sealed-secrets --controller-namespace kube-system --format yaml < secret.yaml > sealed-secret.yaml

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

You have successfully deployed WordPress using encrypted secrets stored in Git and managed by Kubernetes.
