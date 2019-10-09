# FAQ

## CoreDNS Can't Resolve Anything in PODs

Found this issue while trying to play with dns related topics.

```
$ kubectl run --generator=run-pod/v1 tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
If you don't see a command prompt, try pressing enter.
bash-5.0# nc -vz 10.96.0.10 53
nc: connect to 10.96.0.10 port 53 (tcp) failed: Operation timed out
bash-5.0# dig kubernetes.default.svc.cluster.local

; <<>> DiG 9.14.1 <<>> kubernetes.default.svc.cluster.local
;; global options: +cmd
;; connection timed out; no servers could be reached
```

I tried even in controller and worker nodes:

In controller:

```
$ nc -vz 10.96.0.10 53
Connection to 10.96.0.10 53 port [tcp/domain] succeeded!
```

In worker:

```
$ nc -vz 10.96.0.10 53
nc: connect to 10.96.0.10 port 53 (tcp) failed: Connection timed out
```

So I was thinking it might be caused by a couple of potential issues around CNI networking (as I'm using `Canal`), systemd service `systemd-resoved`, and of course `CoreDNS`.

It turned out a restart of CoreDNS pods worked like a charm:

```
kubectl -n kube-system rollout restart deployment coredns
```

The issue had been reported, [here](https://github.com/kubernetes/kubeadm/issues/1731)


## How to update Kubeadm config after `kubeadm init`?

It's common to enable/disable features in a running cluster, after `kubeadm init`.

So the typical steps would be:

1. Update `kubeadm-config.yaml`

For example, in `k8s-controller-0`:

```sh
{
  KUBERNETES_PUBLIC_ADDRESS=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/k8s-public-ip)

  CLUSTER_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/cluster-cidr)

  cat <<EOF | sudo tee /etc/kubernetes/kubeadm/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "${KUBERNETES_PUBLIC_ADDRESS}:6443"
networking:
  podSubnet: "${CLUSTER_CIDR}"
apiServer:
  extraArgs:
    enable-admission-plugins: NodeRestriction,PodSecurityPolicy
    feature-gates: "DynamicAuditing=true"
controllerManager:
  extraArgs:
    cloud-provider: gce
    cloud-config: /etc/kubernetes/cloud
  extraVolumes:
  - name: cloud-config
    hostPath: /etc/kubernetes/cloud-config
    mountPath: /etc/kubernetes/cloud
    pathType: FileOrCreate
EOF
}
```

2. Apply it

Do a dry-run first:

```
$ kubectl get nodes
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-0   Ready    master   52d   v1.15.3
k8s-worker-0       Ready    <none>   52d   v1.15.3
k8s-worker-1       Ready    <none>   52d   v1.15.3
k8s-worker-2       Ready    <none>   52d   v1.15.3

$ sudo kubeadm upgrade apply v1.15.3 \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --cri-socket="/var/run/crio/crio.sock" \
    --dry-run
```

> Note: since I don't want to upgrade the Kubernetes version, so I use the same (e.g. `v1.15.3`)

Apply it if everything is fine:

```
sudo kubeadm upgrade apply v1.15.3 \
    --config=/etc/kubernetes/kubeadm/kubeadm-config.yaml \
    --cri-socket="/var/run/crio/crio.sock"
```

3. Verify it (in a very raw way):

```
$ ps -ef|grep feature
root     15406 15382  5 07:53 ?        00:00:34 kube-apiserver ... feature-gates=DynamicAuditing=true ...
```
