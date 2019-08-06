# Provisioning Infrastructure

To spin up a highly available Kubernetes cluster, we basically need below core infrastructure items:
- 1 x VPC with 1 x Subnet
- 1 x Load Balancer for Kubernetes API Servers
- Some required firewall rules
- 3 x Controller Nodes
- 3 x Worker Nodes

## Networking, Rourting, Firewalling

### Create a Virtual Private Cloud (VPC) Network and Subnet

```sh
{
  gcloud compute networks create kubernetes-the-kubeadm-way --subnet-mode custom

  gcloud compute networks subnets create kubernetes \
    --network kubernetes-the-kubeadm-way \
    --range 10.240.0.0/24
}
```

### Create a Public IP Address for Load Balancer

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

```sh
gcloud compute addresses create kubernetes-the-kubeadm-way \
  --region $(gcloud config get-value compute/region)
```

### Provision a Network Load Balancer

Create the external load balancer network resources:

```sh
{
  KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-kubeadm-way \
    --region $(gcloud config get-value compute/region) \
    --format 'value(address)')

  # health check
  gcloud compute http-health-checks create kubernetes-health-check \
    --description "Kubernetes Health Check" \
    --host "kubernetes.default.svc.cluster.local" \
    --request-path "/healthz"

  # TCP load balancer
  gcloud compute target-pools create kubernetes-the-kubeadm-way-target-pool \
    --http-health-check kubernetes-health-check

  # backend instances: the controllers -- see notes below
  gcloud compute target-pools add-instances kubernetes-the-kubeadm-way-target-pool \
   --instances k8s-controller-0

  # forwarding rules
  gcloud compute forwarding-rules create kubernetes-the-kubeadm-way-forwarding-rule \
    --address ${KUBERNETES_PUBLIC_ADDRESS} \
    --ports 6443 \
    --region $(gcloud config get-value compute/region) \
    --target-pool kubernetes-the-kubeadm-way-target-pool
}
```

> Note: based on what I've tested, we can't add all instances, say `k8s-controller-0`, `k8s-controller-1`, `k8s-controller-2`, to the target-pools at first. Otherwise you may encounter connectivity issues while issuing command of `sudo kubeadm join` in all other Controller nodes other than the first one. Bug or did I miss anything?

### Firewall Rules

Create a firewall rule that allows internal communication across all protocols:

```sh
gcloud compute firewall-rules create kubernetes-the-kubeadm-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-kubeadm-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

Create a firewall rule that allows external TCP, SSH, ICMP:

```sh
gcloud compute firewall-rules create kubernetes-the-kubeadm-way-allow-external \
  --allow tcp:22,tcp:6443,tcp:80,icmp \
  --network kubernetes-the-kubeadm-way \
  --source-ranges 0.0.0.0/0
```

List the firewall rules in the `kubernetes-the-kubeadm-way` VPC network:

```sh
gcloud compute firewall-rules list --filter="network:kubernetes-the-kubeadm-way"
```

> output

```
NAME                                       NETWORK                     DIRECTION  PRIORITY  ALLOW                        DENY  DISABLED
kubernetes-the-kubeadm-way-allow-external  kubernetes-the-kubeadm-way  INGRESS    1000      tcp:22,tcp:6443,tcp:80,icmp        False
kubernetes-the-kubeadm-way-allow-internal  kubernetes-the-kubeadm-way  INGRESS    1000      tcp,udp,icmp                       False
```

## Compute Instances

### Kubernetes Controllers

Create three compute instances which will host the Kubernetes control plane:

```sh
for i in 0 1 2; do
  gcloud compute instances create k8s-controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-2 \
    --metadata cluster-cidr=10.200.0.0/16,k8s-public-ip=${KUBERNETES_PUBLIC_ADDRESS} \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-kubeadm-way,controller
done
```

> Note: using `n1-standard-1` may trigger a warning later as 2 vCPUs are kind of a requirement, so `n1-standard-2` is recommended.

### Kubernetes Workers

Create three compute instances which will host the Kubernetes worker nodes:

```sh
for i in 0 1 2; do
  gcloud compute instances create k8s-worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-kubeadm-way,worker
done
```

Next: [03-Preparing Controllers Before Running `kubeadm`](03-prepare-controllers.md)
