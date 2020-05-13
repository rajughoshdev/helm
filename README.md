# helm v2
Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

Helm v2.x comprises of two parts: a client and a server (Tiller) inside the kube-system namespace. Tiller runs inside your Kubernetes cluster, and manages releases (installations) of your charts. To be able to do this, Tiller needs access to the Kubernetes API. By default, RBAC policies will not allow Tiller to carry out these operations, so we need to do the following:

Create a Service Account tiller for the Tiller server (in the kube-system namespace). As we mentioned before, Service Accounts are meant for intra-cluster processes running in Pods.

Bind the cluster-admin ClusterRole to this Service Account. We will use ClusterRoleBindings as we want it to be applicable in all namespaces. The reason is that we want Tiller to manage resources in all namespaces.

Update the existing Tiller deployment (tiller-deploy) to associate its pod with the Service Account tiller.

The cluster-admin ClusterRole exists by default in your Kubernetes cluster, and allows superuser operations in all of the cluster resources. The reason for binding this role is because with Helm charts, you can have deployments consisting of a wide variety of Kubernetes resources. For instance:

- Pods
- PersistentVolumes
- ConfigMaps
- Deployments
- Secrets
- Namespaces
- Replicasets
- Roles
- RoleBindings

So to make Helm compatible with any existing chart, binding the cluster-admin to the tiller Service Account is the best option. However, if you plan to use a very specific type of Helm chart (for example, one that only creates ConfigMaps, Pods, PersistentVolumes and Secrets), you could create more restrictive RBAC rules.

Step 1: Create the Tiller service account:

Create a tiller-serviceaccount.yaml file using kubectl:
```
kubectl create serviceaccount tiller --namespace kube-system
```
Step 2: Bind the Tiller service account to the cluster-admin role
Create a tiller-clusterrolebinding.yaml file with the following contents:
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
Deploy the ClusterRoleBinding:
```
Create tiller-clusterrolebinding
```
kubectl create -f tiller-clusterrolebinding.yaml
````



Step 3: Install helm client
From Snap (Linux)
```
sudo snap install helm --classic
```
From Homebrew (macOS)
```
brew install kubernetes-helm
```

Step 4: Install Tiller in your cluster

Once you have Helm ready, you can initialize the local CLI and also install Tiller into your Kubernetes cluster in one step:
```
$ helm init --history-max 25
````
TIP: Setting --history-max on helm init is recommended as configmaps and other objects in helm history can grow large in number if not purged by max limit. Without a max history set the history is kept indefinitely, leaving a large number of records for helm and tiller to maintain.

Step 5: Test the helm triller is installed successfully  by using dry-run
```
helm install --name my-release stable/mysql --dry-run --debug
```
Useful link:
If you want to know more about RBAC and user authorization in Kubernetes, take a look at the following links:

[Official Kubernetes documentation for RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

[Official Kubernetes documentation for Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

