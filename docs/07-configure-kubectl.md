# Configuring `kubectl` for Remote Access

Run the commands in this lab from the same directory used to generate the admin client certificates.

## The Admin User

Get back the `admin`:

```sh
gcloud compute scp root@k8s-controller-0:/etc/kubernetes/admin.conf ~/.kube/config
```

## Verification

Check the nodes we have:

```sh
kubectl get no
```

> output

```
NAME               STATUS   ROLES    AGE     VERSION
k8s-controller-0   Ready    master   3h59m   v1.15.1
k8s-controller-1   Ready    master   3h41m   v1.15.1
k8s-controller-2   Ready    master   3h24m   v1.15.1
k8s-worker-0       Ready    <none>   177m    v1.15.1
k8s-worker-1       Ready    <none>   159m    v1.15.1
k8s-worker-2       Ready    <none>   158m    v1.15.1
```

Check the health of the remote Kubernetes cluster:

```sh
kubectl get componentstatuses
```

> output

```
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

Check all pods we have:

```sh
kubectl get po --all-namespaces -o wide
```

> output

```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE               NOMINATED NODE   READINESS GATES
default       nginx-7bb7cd8db5-2l7j4                     1/1     Running   0          15h   10.200.4.6    k8s-worker-1       <none>           <none>
default       nginx-7bb7cd8db5-2psm7                     1/1     Running   0          15h   10.200.5.6    k8s-worker-2       <none>           <none>
default       nginx-7bb7cd8db5-ktr85                     1/1     Running   0          15h   10.200.3.7    k8s-worker-0       <none>           <none>
kube-system   canal-6hwtr                                2/2     Running   0          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   canal-99jch                                2/2     Running   0          18h   10.240.0.20   k8s-worker-0       <none>           <none>
kube-system   canal-9chtk                                2/2     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
kube-system   canal-drvgr                                2/2     Running   0          18h   10.240.0.21   k8s-worker-1       <none>           <none>
kube-system   canal-nmv6c                                2/2     Running   0          18h   10.240.0.22   k8s-worker-2       <none>           <none>
kube-system   canal-vgzvd                                2/2     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   coredns-5c98db65d4-8knst                   1/1     Running   0          19h   10.88.0.2     k8s-controller-0   <none>           <none>
kube-system   coredns-5c98db65d4-zxhqf                   1/1     Running   0          19h   10.88.0.3     k8s-controller-0   <none>           <none>
kube-system   etcd-k8s-controller-0                      1/1     Running   0          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   etcd-k8s-controller-1                      1/1     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   etcd-k8s-controller-2                      1/1     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
kube-system   kube-apiserver-k8s-controller-0            1/1     Running   0          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   kube-apiserver-k8s-controller-1            1/1     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   kube-apiserver-k8s-controller-2            1/1     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
kube-system   kube-controller-manager-k8s-controller-0   1/1     Running   1          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   kube-controller-manager-k8s-controller-1   1/1     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   kube-controller-manager-k8s-controller-2   1/1     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
kube-system   kube-proxy-4mns5                           1/1     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   kube-proxy-8nwzd                           1/1     Running   0          18h   10.240.0.21   k8s-worker-1       <none>           <none>
kube-system   kube-proxy-9ck2j                           1/1     Running   0          18h   10.240.0.20   k8s-worker-0       <none>           <none>
kube-system   kube-proxy-fnbxg                           1/1     Running   0          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   kube-proxy-m4lfb                           1/1     Running   0          18h   10.240.0.22   k8s-worker-2       <none>           <none>
kube-system   kube-proxy-wqkdh                           1/1     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
kube-system   kube-scheduler-k8s-controller-0            1/1     Running   1          19h   10.240.0.10   k8s-controller-0   <none>           <none>
kube-system   kube-scheduler-k8s-controller-1            1/1     Running   0          19h   10.240.0.11   k8s-controller-1   <none>           <none>
kube-system   kube-scheduler-k8s-controller-2            1/1     Running   0          19h   10.240.0.12   k8s-controller-2   <none>           <none>
```

Next: [08-Performing Smoke Tests](08-smoke-test.md)
