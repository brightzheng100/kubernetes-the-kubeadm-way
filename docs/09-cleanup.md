# Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances

Delete the controller and worker compute instances:

```sh
gcloud -q compute instances delete \
  k8s-controller-0 k8s-controller-1 k8s-controller-2 \
  k8s-worker-0 k8s-worker-1 k8s-worker-2
```

## Networking, Routing, Firewalling

Delete the external load balancer network resources:

```sh
{
  gcloud -q compute forwarding-rules delete kubernetes-the-kubeadm-way-forwarding-rule \
    --region $(gcloud config get-value compute/region)

  gcloud -q compute target-pools delete kubernetes-the-kubeadm-way-target-pool

  gcloud -q compute http-health-checks delete kubernetes-health-check

  gcloud -q compute addresses delete kubernetes-the-kubeadm-way
}
```

Delete the `kubernetes-the-hard-way` firewall rules:

```sh
gcloud -q compute firewall-rules delete \
  kubernetes-the-kubeadm-way-allow-nginx-service \
  kubernetes-the-kubeadm-way-allow-internal \
  kubernetes-the-kubeadm-way-allow-external \
  kubernetes-the-kubeadm-way-allow-health-check
```

Delete the `kubernetes-the-hard-way` network VPC:

```
{
  gcloud -q compute routes delete \
    kubernetes-route-10-200-0-0-24 \
    kubernetes-route-10-200-1-0-24 \
    kubernetes-route-10-200-2-0-24

  gcloud -q compute networks subnets delete kubernetes

  gcloud -q compute networks delete kubernetes-the-kubeadm-way
}
```
