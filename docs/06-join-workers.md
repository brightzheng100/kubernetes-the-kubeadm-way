# Joining Workers

## The First Worker

SSH into the first Worker node: `k8s-worker-0`:

```sh
sudo kubeadm join 35.247.46.244:6443 \
  --token ivtu7b.kjfnq32mewyzz1cd \
  --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032
```

You can verify whether it's joined the cluster by running below command in any of the Controller nodes.
For example, you may run it in `k8s-controller-0`:

```sh
kubectl get no
```

> output

```
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-0   Ready    master   62m   v1.15.1
k8s-controller-1   Ready    master   44m   v1.15.1
k8s-controller-2   Ready    master   27m   v1.15.1
k8s-worker-0       Ready    <none>   58s   v1.15.1
```

## The Second Worker

SSH into the second Worker node: `k8s-worker-1`:

```sh
sudo kubeadm join 35.247.46.244:6443 \
  --token ivtu7b.kjfnq32mewyzz1cd \
  --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032
```

You can verify whether it's joined the cluster by running below command in any of the Controller nodes.
For example, you may run it in `k8s-controller-0`:

```sh
kubectl get no
```

> output

```
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-0   Ready    master   79m   v1.15.1
k8s-controller-1   Ready    master   61m   v1.15.1
k8s-controller-2   Ready    master   44m   v1.15.1
k8s-worker-0       Ready    <none>   17m   v1.15.1
k8s-worker-1       Ready    <none>   16s   v1.15.1
```

## The Third Worker

SSH into the third Worker node: `k8s-worker-2`:

```sh
sudo kubeadm join 35.247.46.244:6443 \
  --token ivtu7b.kjfnq32mewyzz1cd \
  --discovery-token-ca-cert-hash sha256:4aec11ae9194ff0494ad3c54ad782ccf85001f17659cad8698b0bd8cf7f48032
```

You can verify whether it's joined the cluster by running below command in any of the Controller nodes.
For example, you may run it in `k8s-controller-0`:

```sh
kubectl get no
```

> output

```
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-0   Ready    master   81m   v1.15.1
k8s-controller-1   Ready    master   62m   v1.15.1
k8s-controller-2   Ready    master   46m   v1.15.1
k8s-worker-0       Ready    <none>   19m   v1.15.1
k8s-worker-1       Ready    <none>   94s   v1.15.1
k8s-worker-2       Ready    <none>   6s    v1.15.1
```

> Note: you of course can join more workers, as needed, by following exactly the same steps.


Next: [07-Configuring `kubectl` for Remote Access](07-configure-kubectl.md)
