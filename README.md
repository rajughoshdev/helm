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


## Enable Helm in your cluster

### INITIALIZE HELM AND INSTALL TILLER
Once you have Helm ready, you can initialize the local CLI and also install Tiller into your Kubernetes cluster in one step:
```
$ helm init --history-max 25
````
TIP: Setting --history-max on helm init is recommended as configmaps and other objects in helm history can grow large in number if not purged by max limit. Without a max history set the history is kept indefinitely, leaving a large number of records for helm and tiller to maintain.

