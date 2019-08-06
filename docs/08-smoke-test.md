# Performing Smoke Tests

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```sh
kubectl create secret generic kubernetes-the-kubeadm-way \
  --from-literal="mykey=mydata"
```

Print a hexdump of the `kubernetes-the-kubeadm-way` secret stored in etcd:

```sh
kubectl exec po/etcd-k8s-controller-0 -n kube-system -- /bin/sh -ec \
  "ETCDCTL_API=3 etcdctl \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  get /registry/secrets/default/kubernetes-the-kubeadm-way | hexdump -C"
```

> output

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 6b 75 62 65 61 64 6d  |etes-the-kubeadm|
00000030  2d 77 61 79 0a 6b 38 73  00 0a 0c 0a 02 76 31 12  |-way.k8s.....v1.|
00000040  06 53 65 63 72 65 74 12  7a 0a 5f 0a 1a 6b 75 62  |.Secret.z._..kub|
00000050  65 72 6e 65 74 65 73 2d  74 68 65 2d 6b 75 62 65  |ernetes-the-kube|
00000060  61 64 6d 2d 77 61 79 12  00 1a 07 64 65 66 61 75  |adm-way....defau|
00000070  6c 74 22 00 2a 24 38 64  36 65 38 34 32 34 2d 33  |lt".*$8d6e8424-3|
00000080  62 64 30 2d 34 33 38 31  2d 39 61 62 38 2d 32 64  |bd0-4381-9ab8-2d|
00000090  66 30 62 31 38 62 35 33  66 39 32 00 38 00 42 08  |f0b18b53f92.8.B.|
000000a0  08 e7 d0 a0 ea 05 10 00  7a 00 12 0f 0a 05 6d 79  |........z.....my|
000000b0  6b 65 79 12 06 6d 79 64  61 74 61 1a 06 4f 70 61  |key..mydata..Opa|
000000c0  71 75 65 1a 00 22 00 0a                           |que.."..|
000000c8
```

> Result: Obviously, the `kubeadm` process I did doesn't enable the data encryption, yet.

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```sh
kubectl run nginx --image=nginx --replicas=3
```

List the pod created by the `nginx` deployment:

```sh
kubectl get po -l run=nginx -o wide
```

> output

```
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx-7bb7cd8db5-2l7j4   1/1     Running   0          26m   10.200.4.6   k8s-worker-1   <none>           <none>
nginx-7bb7cd8db5-2psm7   1/1     Running   0          26m   10.200.5.6   k8s-worker-2   <none>           <none>
nginx-7bb7cd8db5-ktr85   1/1     Running   0          26m   10.200.3.7   k8s-worker-0   <none>           <none>
```

> Result: the pods are properly distributed to Worker nodes.

### Communication Between Pods

In this section you will verify the ability to communicate from one pod to another, whichever node they're deployed.

```sh
kubectl run --generator=run-pod/v1 tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```

Try `ping` through all `nginx` pods' IPs:

```
bash-5.0# ping -c 1 10.200.4.6
PING 10.200.4.6 (10.200.4.6) 56(84) bytes of data.
64 bytes from 10.200.4.6: icmp_seq=1 ttl=62 time=1.10 ms

--- 10.200.4.6 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.104/1.104/1.104/0.000 ms
bash-5.0# ping -c 1 10.200.5.6
PING 10.200.5.6 (10.200.5.6) 56(84) bytes of data.
64 bytes from 10.200.5.6: icmp_seq=1 ttl=63 time=0.092 ms

--- 10.200.5.6 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.092/0.092/0.092/0.000 ms
bash-5.0# ping -c 1 10.200.3.7
PING 10.200.3.7 (10.200.3.7) 56(84) bytes of data.
64 bytes from 10.200.3.7: icmp_seq=1 ttl=62 time=1.80 ms

--- 10.200.3.7 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.797/1.797/1.797/0.000 ms
```

> Result: yes, they're ping-able.

And `curl` all `nginx` endpoint:

```
bash-5.0# curl --head 10.200.4.6
HTTP/1.1 200 OK
Server: nginx/1.17.2
Date: Mon, 05 Aug 2019 13:32:14 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 23 Jul 2019 11:45:37 GMT
Connection: keep-alive
ETag: "5d36f361-264"
Accept-Ranges: bytes

bash-5.0# curl --head 10.200.5.6
HTTP/1.1 200 OK
Server: nginx/1.17.2
Date: Mon, 05 Aug 2019 13:32:19 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 23 Jul 2019 11:45:37 GMT
Connection: keep-alive
ETag: "5d36f361-264"
Accept-Ranges: bytes

bash-5.0# curl --head 10.200.3.7
HTTP/1.1 200 OK
Server: nginx/1.17.2
Date: Mon, 05 Aug 2019 13:32:24 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 23 Jul 2019 11:45:37 GMT
Connection: keep-alive
ETag: "5d36f361-264"
Accept-Ranges: bytes
```

> Result: yes, they're curl-able too.

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```sh
POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```sh
kubectl port-forward $POD_NAME 8080:80
```

> output

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.17.2
Date: Mon, 05 Aug 2019 13:35:35 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 23 Jul 2019 11:45:37 GMT
Connection: keep-alive
ETag: "5d36f361-264"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

> Result: `port-forward` works.

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```sh
kubectl logs $POD_NAME
```

> output

```
127.0.0.1 - - [05/Aug/2019:13:35:21 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.54.0" "-"
```

> Result: `kubectl logs` works.

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

> output

```
nginx version: nginx/1.17.2
```

> Result: `kubectl exec` works.

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```sh
kubectl expose deployment nginx --port 80 --type NodePort
```

Retrieve the node port assigned to the `nginx` service:

```sh
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
echo $NODE_PORT
```

> output

```
30722
```

Create a firewall rule that allows remote access to the `nginx` node port:

```sh
gcloud compute firewall-rules create kubernetes-the-kubeadm-way-allow-nginx-service \
  --allow=tcp:${NODE_PORT} \
  --network kubernetes-the-kubeadm-way
```

Retrieve the external IP address of a worker instance:

```sh
EXTERNAL_IP=$(gcloud compute instances describe k8s-worker-0 \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')
```

Make an HTTP request using the external IP address and the `nginx` node port:

```sh
curl -I "http://${EXTERNAL_IP}:${NODE_PORT}"
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.17.2
Date: Mon, 05 Aug 2019 13:40:03 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 23 Jul 2019 11:45:37 GMT
Connection: keep-alive
ETag: "5d36f361-264"
Accept-Ranges: bytes
```

> Result: Exposing pods to external world by `kubectl service` works.

Next: [09-Cleaning Up](09-cleanup.md)
