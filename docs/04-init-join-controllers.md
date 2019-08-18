# Bootstrapping And Joining Controllers

## Bootstrapping The First Controller

Now let's focus on our first Controller node: `k8s-controller-0`.
SSH into it: `gcloud compute ssh k8s-controller-0`

### Run `kubeadm init`

```sh
{
  KUBERNETES_PUBLIC_ADDRESS=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/k8s-public-ip)

  CLUSTER_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/cluster-cidr)

  sudo mkdir /etc/kubernetes/kubeadm/
  cat <<EOF | sudo tee /etc/kubernetes/kubeadm/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "${KUBERNETES_PUBLIC_ADDRESS}:6443"
networking:
  podSubnet: "${CLUSTER_CIDR}"
EOF

  sudo kubeadm init \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --cri-socket="/var/run/crio/crio.sock" \
    --upload-certs
}
```

> output:

```
[init] Using Kubernetes version: v1.15.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-controller-0 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.$
luster.local] and IPs [10.96.0.1 10.240.0.10 35.247.46.244]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-controller-0 localhost] and IPs [10.240.0.10 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-controller-0 localhost] and IPs [10.240.0.10 127.0.0.1 ::1]
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 21.002868 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
571b1177773d5209296e910155af31c4842ed3ae74bc78a210b54e36d17b9d29
[mark-control-plane] Marking the node k8s-controller-0 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-controller-0 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: ivtu7b.kjfnq32mewyzz1cd
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 35.247.46.244:6443 --token ivtu7b.kjfnq32mewyzz1cd \
    --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032 \
    --control-plane --certificate-key 571b1177773d5209296e910155af31c4842ed3ae74bc78a210b54e36d17b9d29

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 35.247.46.244:6443 --token ivtu7b.kjfnq32mewyzz1cd \
    --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032
```

> Notes: 
> 1. don't try to **hack** into this cluster as it's been shutdown immediately after this tutorial.
> 2. use `sudo kubeadm reset --cri-socket="/var/run/crio/crio.sock"` to reset it if you want to do it again after fixes.

### Post Actions

#### Configure `kubectl`

```
{
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
}
```

Verify it:

```
$ kubectl get no
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-0   Ready    master   62m   v1.15.1

$ kubectl get po --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-5c98db65d4-8knst                   1/1     Running   0          8m17s
kube-system   coredns-5c98db65d4-zxhqf                   1/1     Running   0          8m17s
kube-system   etcd-k8s-controller-0                      1/1     Running   0          7m13s
kube-system   kube-apiserver-k8s-controller-0            1/1     Running   0          7m15s
kube-system   kube-controller-manager-k8s-controller-0   1/1     Running   0          7m16s
kube-system   kube-proxy-fnbxg                           1/1     Running   0          8m16s
kube-system   kube-scheduler-k8s-controller-0            1/1     Running   0          7m36s
```

#### Installing a pod network add-on

There are many CNI providers but we'll try `Canal` in this tutorial.

Deploy `Canal` CNI plugin:

```sh
{
  curl https://docs.projectcalico.org/v3.8/manifests/canal.yaml -O

  CLUSTER_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/cluster-cidr)
  sed -i -e "s?10.244.0.0/16?$CLUSTER_CIDR?g" canal.yaml

  kubectl apply -f canal.yaml
}
```

> Output

```
...
configmap/canal-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/canal-flannel created
clusterrolebinding.rbac.authorization.k8s.io/canal-calico created
daemonset.apps/canal created
serviceaccount/canal created
```

Wait for a while and check it again:

```
$ kubectl get po --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   canal-7z425                                2/2     Running   0          20m
kube-system   coredns-5c98db65d4-gb6m8                   1/1     Running   0          35m
kube-system   coredns-5c98db65d4-zbw8g                   1/1     Running   0          35m
kube-system   etcd-k8s-controller-0                      1/1     Running   0          34m
kube-system   kube-apiserver-k8s-controller-0            1/1     Running   0          34m
kube-system   kube-controller-manager-k8s-controller-0   1/1     Running   0          34m
kube-system   kube-proxy-dwskx                           1/1     Running   0          35m
kube-system   kube-scheduler-k8s-controller-0            1/1     Running   0          35m
```

> Official guide is [here](https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/flannel)


## Joining The Rest of Controllers

### The Second Controller

SSH into the second Controller node: `k8s-controller-1`:

```sh
sudo kubeadm join 35.247.46.244:6443 --token ivtu7b.kjfnq32mewyzz1cd \
    --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032 \
    --control-plane --certificate-key 571b1177773d5209296e910155af31c4842ed3ae74bc78a210b54e36d17b9d29
```

> Note: this command can be retrieved from the output while performing `kubeadm init` in `k8s-controller-0`.

Similarly to the first Controller node:

```sh
{
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
}
```

Verify it:

```sh
kubectl get no
```

> output

```
NAME               STATUS   ROLES    AGE     VERSION
k8s-controller-0   Ready    master   25m     v1.15.1
k8s-controller-1   Ready    master   6m43s   v1.15.1
```

```sh
kubectl get po --all-namespaces
```

> output

```
NAMESPACE     NAME                                       READY   STATUS                   RESTARTS   AGE
kube-system   canal-6hwtr                                2/2     Running                  0          15m
kube-system   canal-vgzvd                                0/2     Init:ImageInspectError   0          7m7s
kube-system   coredns-5c98db65d4-8knst                   1/1     Running                  0          25m
kube-system   coredns-5c98db65d4-zxhqf                   1/1     Running                  0          25m
kube-system   etcd-k8s-controller-0                      1/1     Running                  0          24m
kube-system   etcd-k8s-controller-1                      1/1     Running                  0          7m7s
kube-system   kube-apiserver-k8s-controller-0            1/1     Running                  0          24m
kube-system   kube-apiserver-k8s-controller-1            1/1     Running                  0          5m45s
kube-system   kube-controller-manager-k8s-controller-0   1/1     Running                  1          24m
kube-system   kube-controller-manager-k8s-controller-1   1/1     Running                  0          5m50s
kube-system   kube-proxy-4mns5                           1/1     Running                  0          7m7s
kube-system   kube-proxy-fnbxg                           1/1     Running                  0          25m
kube-system   kube-scheduler-k8s-controller-0            1/1     Running                  1          24m
kube-system   kube-scheduler-k8s-controller-1            1/1     Running                  0          5m53s
```

### The Third Controller

SSH into `k8s-controller-2`, issue the same `kubeadm join` command:

```sh
sudo kubeadm join 35.247.46.244:6443 --token ivtu7b.kjfnq32mewyzz1cd \
    --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032 \
    --control-plane --certificate-key 571b1177773d5209296e910155af31c4842ed3ae74bc78a210b54e36d17b9d29
```

Similarly:

```sh
{
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
}
```

Verify it:

```sh
kubectl get no
```

> output
 
```
NAME               STATUS   ROLES    AGE    VERSION
k8s-controller-0   Ready    master   37m    v1.15.1
k8s-controller-1   Ready    master   18m    v1.15.1
k8s-controller-2   Ready    master   2m5s   v1.15.1
```

```sh
kubectl get po --all-namespaces
```

> output

```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   canal-6hwtr                                2/2     Running   0          29m
kube-system   canal-9chtk                                2/2     Running   0          4m57s
kube-system   canal-vgzvd                                2/2     Running   0          21m
kube-system   coredns-5c98db65d4-8knst                   1/1     Running   0          39m
kube-system   coredns-5c98db65d4-zxhqf                   1/1     Running   0          39m
kube-system   etcd-k8s-controller-0                      1/1     Running   0          38m
kube-system   etcd-k8s-controller-1                      1/1     Running   0          21m
kube-system   etcd-k8s-controller-2                      1/1     Running   0          4m53s
kube-system   kube-apiserver-k8s-controller-0            1/1     Running   0          38m
kube-system   kube-apiserver-k8s-controller-1            1/1     Running   0          20m
kube-system   kube-apiserver-k8s-controller-2            1/1     Running   0          4m57s
kube-system   kube-controller-manager-k8s-controller-0   1/1     Running   1          38m
kube-system   kube-controller-manager-k8s-controller-1   1/1     Running   0          20m
kube-system   kube-controller-manager-k8s-controller-2   1/1     Running   0          4m57s
kube-system   kube-proxy-4mns5                           1/1     Running   0          21m
kube-system   kube-proxy-fnbxg                           1/1     Running   0          39m
kube-system   kube-proxy-wqkdh                           1/1     Running   0          4m57s
kube-system   kube-scheduler-k8s-controller-0            1/1     Running   1          39m
kube-system   kube-scheduler-k8s-controller-1            1/1     Running   0          20m
kube-system   kube-scheduler-k8s-controller-2            1/1     Running   0          4m57s
```

### Add These Controller Nodes To Load Balancer

Run this command in the machine where you spun up the infrastructure, instead of any of Controller nodes.

```sh
gcloud compute target-pools add-instances kubernetes-the-kubeadm-way-target-pool \
  --instances k8s-controller-1,k8s-controller-2
```

And verify it:

```sh
gcloud compute target-pools describe kubernetes-the-kubeadm-way-target-pool --format=json | jq .instances
```

> output

```
[
  "https://www.googleapis.com/compute/v1/projects/XXX/zones/us-west1-b/instances/k8s-controller-0",
  "https://www.googleapis.com/compute/v1/projects/XXX/zones/us-west1-b/instances/k8s-controller-1",
  "https://www.googleapis.com/compute/v1/projects/XXX/zones/us-west1-b/instances/k8s-controller-2"
]
```


Next: [05-Prepare Workers Before Running `kubeadm`](05-prepare-workers.md)
