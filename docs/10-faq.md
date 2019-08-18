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
