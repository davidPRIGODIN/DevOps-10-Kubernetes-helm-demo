# DevOps-10-Kubernetes-helm-demo

This demo project demonstrates how to deploy a stateful application on Kubernetes using Helm.

Helm is a package manager for Kubernetes. It makes it easy to deploy, configure, and manage applications in a Kubernetes cluster.

## 1. Create K8s Cluster on Linode

Go to the [Linode Cloud Manager](https://cloud.linode.com/) and create a Kubernetes cluster.

* Linode → Compute → Kubernetes →  Create Cluster
* Cluster Label: test
* Region: Choose your closest region
* Kubernetes Version: Choose the latest
* Akamai App Platform: No
* HA Control Plane: No
* Dedicated 4GB → Configure Pool → Nodes: 2
* Create Cluster

Download test-kubeconfig.yaml:
 ```bash
chmod 400 test-kubeconfig.yaml
mv test-kubeconfig.yaml ~/helm-demo
```

Set this Kubeconfig as env variable:
```bash
export KUBECONFIG=test-kubeconfig.yaml
kubectl get node
```

We are now connected to the remote cluster.

## 2. Deploy MongoDB StatefulSet

### 2.1. Install Helm

Follow the official [Helm installation guide](https://helm.sh/docs/intro/install/).

The MongoDB Helm chart is maintained by Bitnami.

Add the Bitnami Helm repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Search for the available MongoDB chart:

```bash
helm search repo bitnami/mongodb
```

### 2.2. Install MongoDB

Install MongoDB in your Kubernetes cluster, overriding the chart's default values with custom configurations defined in `helm-mongodb.yaml`:

```bash
helm install mongodb --values helm-mongodb.yaml bitnami/mongodb
```

Verify that the pods and secrets have been created:

```bash
kubectl get pods
kubectl get secrets
```

### 2.3. Verify the Deployment

Three MongoDB pods have been successfully deployed, as configured.

Additionally, in **Linode → Storage → Volumes**, we can see that three persistent volumes have been provisioned.

Our MongoDB StatefulSet is now running in the Kubernetes cluster.

## 3. Deploy Mongo Express

Deploy Mongo Express in the Kubernetes cluster using the following command and verify that the pod is running:

```bash
kubectl apply -f helm-mongo-express.yaml
kubectl get pods
```
Mongo Express is now deployed and ready to provide a web-based interface for managing MongoDB.

## 4. Ingress

### 4.1. Deploy Ingress Controller

Install the NGINX Ingress Controller using Helm and verify that the pods are running:

```bash
helm install nginx-ingress oci://ghcr.io/nginx/charts/nginx-ingress --set controller.reportIngressStatus.enable=true
kubectl get pods
```

In **Linode → Networking → NodeBalancers**, we can verify that a NodeBalancer has been successfully created.

### 4.2. Create Ingress Rule

Create an Ingress resource to define how external traffic reaches the application within the Kubernetes cluster:

```bash
kubectl apply -f helm-ingress.yaml
kubectl get ingress
```

### 4.3. Access Mongo Express via the NodeBalancer Hostname

Navigate to **Linode → NodeBalancers → test → Summary** and copy the hostname into your browser's address bar.

We can now create databases and collections.

So everything works.

## 5. Uninstall and Clean Up

### 5.1. Uninstall MongoDB Helm Chart

List the installed Helm releases and uninstall MongoDB:

```bash
helm list
helm uninstall mongodb
```

### 5.2. Delete Linode Resources

Once you are done with the Kubernetes cluster, delete the remaining Linode resources to avoid unnecessary costs:

- **Kubernetes Cluster:** Linode → Compute → Kubernetes → Select `test` → Delete
- **NodeBalancer:** Linode → Networking → NodeBalancers → Select `test` → Delete
- **Persistent Volumes:** Linode → Storage → Volumes → Delete the associated volumes

## Acknowledgements

This demo project was created as part of the DevOps Bootcamp by **TechWorld with Nana**.<br>
Many thanks to Nana for creating such a comprehensive and practical learning experience.
