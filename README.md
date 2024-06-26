# Task: Create User in kubernetes which can only deploy pod to a namespace called deployment-A, user can have read access to every other namespace but cannot modify anything in the other namespaces


# Kubernetes RBAC Guide

This guide provides an overview of Kubernetes Role-Based Access Control (RBAC) concepts and a step-by-step example of setting up RBAC in a Kubernetes cluster.

## What is Kubernetes RBAC?

RBAC (Role-Based Access Control) is a method of limiting access to computer systems by assigning granular roles to users. Each role represents a set of actions the user is allowed to perform, such as creating Kubernetes Pods but not deleting them.

Kubernetes includes a robust RBAC implementation that can be used to segregate users in your cluster. You can set up RBAC rules to restrict users to just the cluster resources they need to access.

## RBAC vs ABAC

RBAC is one kind of access control. ABAC (Attribute-Based Access Control) provides more granularity in sensitive scenarios by considering attributes of the user and target resource. However, Kubernetes only natively supports RBAC-based authorization.

## How RBAC Works in Kubernetes

The Kubernetes API server has built-in RBAC support that works with both Users and Service Accounts. RBAC is configured in your cluster by enabling the RBAC feature and creating objects using the resources provided by the `rbac.authorization.k8s.io` API. These objects include:

- **Role**: A collection of permissions for specific actions on Kubernetes resources within a namespace.
- **ClusterRole**: Similar to Role but for cluster-level resources or globally across all namespaces.
- **RoleBinding**: Links Roles to Users or Service Accounts within a namespace.
- **ClusterRoleBinding**: Links ClusterRoles to Users or Service Accounts at the cluster level.

## Users vs Service Accounts

- **Users**: Represents humans who authenticate to the cluster using external services. Managed outside Kubernetes.
- **Service Accounts**: Token values used to grant access to namespaces in your cluster. Managed within Kubernetes.

## Example: Setting Up RBAC in Kubernetes

This example demonstrates how to create a Service Account, assign it a Role, and verify the permissions.

### 1. Check if RBAC is Enabled

Run the following command to check if RBAC is enabled:

```bash
$ kubectl api-versions | grep rbac
```

To manually enable RBAC, start the Kubernetes API server with the `--authorization-mode=RBAC` flag:

```bash
$ kube-apiserver --authorization-mode=RBAC
```

### 2. Create a Service Account

Create a Service Account:

```bash
$ kubectl create serviceaccount demo-user
```

Generate an authorization token:

```bash
$ TOKEN=$(kubectl create token demo-user)
```

### 3. Configure kubectl with the Service Account

Add the Service Account as a credential:

```bash
$ kubectl config set-credentials demo-user --token=$TOKEN
```

Create a new context for the Service Account:

```bash
$ kubectl config set-context demo-user-context --cluster=default --user=demo-user
$ kubectl config use-context demo-user-context
```

### 4. Create a User

Generate certificates for the user:

```bash
$ openssl genpkey -out malik.key -algorithm Ed25519
$ openssl req -new -key malik.key -out malik.csr -subj "/CN=malik/O=dev"
$ sudo openssl x509 -req -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -days 730 -in malik.csr -out malik.crt
```

Set credentials:

```bash
$ kubectl config set-credentials malik --client-certificate=malik.crt --client-key=malik.key
```

Create a context for the user:

```bash
$ kubectl config set-context malik-kube --cluster=minikube --user=malik --namespace=default
$ kubectl config use-context malik-kube
```

### 5. Create a Role

Create a Role and a ClusterRole:

```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: deployment-a
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get", "watch", "list", "patch", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: other-reader
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["get", "watch", "list"]
```

Apply the Role manifest:

```bash
$ kubectl apply -f role.yaml
```

### 6. Create a RoleBinding

Create a RoleBinding and a ClusterRoleBinding:

```yaml
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: deployment-a
subjects:
- kind: User
  name: malik
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-other-global
subjects:
- kind: User
  name: malik
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: other-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the RoleBinding manifest:

```bash
$ kubectl apply -f rolebinding.yaml
```

### 7. Verify Permissions

Switch to the context that authenticates as the user:

```bash
$ kubectl config use-context malik-kube
```

Verify the permissions:

```bash
$ kubectl get pods
No resources found in default namespace.

$ kubectl auth can-i create deployments --namespace=default --as=malik
no

$ kubectl auth can-i create secrets --namespace=default --as=malik
no

$ kubectl auth can-i get pods --namespace=default --as=malik
yes
```

## Conclusion

You have successfully set up Kubernetes RBAC and created a user with specific permissions. This guide covered the basics of RBAC, including creating roles and bindings, and verifying permissions.
