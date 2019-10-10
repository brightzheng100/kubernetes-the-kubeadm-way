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

Let's take eanbling [Kubernetes Audit](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/) as an example, the typical steps would be:

**1. Update `kubeadm-config.yaml`**

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
    # newly added config
    feature-gates: "DynamicAuditing=true"
    audit-dynamic-configuration: "true"
    runtime-config: "auditregistration.k8s.io/v1alpha1=true"
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

**2. Apply it**

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

**3. Verify it**

In a very raw way:

```
$ ps -ef|grep feature
root     15406 15382  5 07:53 ?        00:00:34 kube-apiserver ... feature-gates=DynamicAuditing=true ...
```

Or you can use Falco's [Helm Chart](https://github.com/helm/charts/blob/master/stable/falco/README.md) to try through -- this really works.


## Error: `failed to ensure load balancer` in Kubernetes on GCE

If you want to expose your pods through `LoadBalancer`, like what [guestbook](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/) example does, you may encounter issues on GCE:

```
$ kubectl describe svc frontend
...
Events:
  Type     Reason                  Age   From                Message
  ----     ------                  ----  ----                -------
  Normal   EnsuringLoadBalancer    9s    service-controller  Ensuring load balancer
  Warning  SyncLoadBalancerFailed  1s    service-controller  Error syncing load balancer: failed to ensure load balancer: no node tags supplied and also failed to parse the given lists of hosts for tags. Abort creating firewall rule
```

The message is super confusing!

The solution is simple: make sure you've tagged each of you nodes with its `hostname`.
For example, edit VM `k8s-worker-0` with a network tag named `k8s-worker-0`.
