# task: Create a second namespace Called deployment B. and Deploy a nginx container. Use Network Policies to ensure that the Nginx container cannot communicate with the wordpress instance using the Kubernetes service name.

This project sets up a Kubernetes environment with two namespaces, deploys an Nginx container in the `deployment-b` namespace, and configures network policies to ensure that the Nginx container cannot communicate with a WordPress instance in the `default` namespace using its Kubernetes service name.

## Prerequisites

- Kubernetes cluster
- install network policy to the cluster 
- kubectl configured to interact with the cluster

How to create a Network Policy in Kubernetes
Before following this guide, you’ll need access to a Kubernetes cluster that’s using a CNI plugin with Network Policy support. If you need to create one, you can start a Minikube cluster and opt-in to using [official Calico]([https://github.com/kubernetes/minikube/releases](https://docs.tigera.io/calico/latest/getting-started/kubernetes/minikube)).


## Steps

### 1. Create `deployment-b` Namespace

Create a new namespace called `deployment-b`.

```bash
kubectl create namespace deployment-b
```

### 2. Deploy Nginx Container in `deployment-b` Namespace

Create a pod for the Nginx container:

    kubectl run pod1 --image nginx:latest -l app=pod1

### 3. Apply Network Policies

#### Network Policy for WordPress Instance

Create a Network Policy in the `deploymnet-b` namespace to restrict access to the WordPress instance.

`network-policy.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy
  namespace: deployment-b
spec:
  podSelector:
    matchLabels:
      app: pod1
  policyTypes:
    - Ingress
```

Apply the network policy:

```bash
kubectl apply -f network-policy.yaml
```

Now run a command inside wordpress to verify that it can communicate with pod1

    kubectl exec -it <wordpress-pod> -- curl <nginx-pod-ip> --max-time 1
    curl: (28) Connection timed out after 1001 milliseconds
    command terminated with exit code 28

The connection does not succeed. The Network Policy targeting wordpress blocks all network traffic so pod1 cannot communicate.

## Conclusion

These steps ensure that the Nginx container deployed in the `deployment-b` namespace cannot communicate with the WordPress instance in the `deployment-a` namespace using its Kubernetes service name, enhancing the security and isolation between the two deployments.
```


