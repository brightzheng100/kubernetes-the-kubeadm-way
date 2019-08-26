# Preparing Controllers

## SSH Into Controllers

You're encouraged to use multi-pane env, like [tmux](https://github.com/tmux/tmux/wiki) or [iTerm2](https://www.iterm2.com/), so you can run below steps in parallel for each Controller.
- In pane 1, SSH into `k8s-controller-0`: `gcloud compute ssh k8s-controller-0`
- In pane 2, SSH into `k8s-controller-1`: `gcloud compute ssh k8s-controller-1`
- In pane 3, SSH into `k8s-controller-2`: `gcloud compute ssh k8s-controller-2`


## Installing Container Runtime: `oci-o`

Prerequisites:

```sh
{
  sudo modprobe overlay
  sudo modprobe br_netfilter

  # Setup required sysctl params, these persist across reboots.
  cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

  sudo sysctl --system
}
```

Then install ans start it up:

```sh
{
  sudo apt-get update
  sudo apt-get install -y software-properties-common

  sudo add-apt-repository -y ppa:projectatomic/ppa
  sudo apt-get update

  sudo apt-get install -y cri-o-1.13
}
```

We need to update the registries in `cri-o` to make sure `quay.io` is whitelisted:

```sh
sudo vi /etc/crio/crio.conf
----
...
[crio.image]
...
registries = [
        "quay.io",
        "docker.io",
]
```

Restart `cri-o`:

```sh
sudo systemctl restart crio
```

## Installing `kubelet`, `kubeadm` and `kubectl`

```
{
  sudo apt-get install -y apt-transport-https curl

  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

  cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
}
```

As we use `cri-o`, we have to edit the config to make sure:
1. specify `cgroup-driver` as `systemd`
2. and add this variable to `ExecStart`

```sh
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
# edited by adding: --cloud-provider=gce --cloud-config=/etc/kubernetes/cloud-config
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --cloud-provider=gce --cloud-config=/etc/kubernetes/cloud-config"
# newly added
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
#ExecStart=/usr/bin/kubelet \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS \$KUBELET_EXTRA_ARGS
ExecStart=/usr/bin/kubelet \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS \$KUBELET_EXTRA_ARGS \$KUBELET_CGROUP_ARGS
EOF
```

And we need to add `cloud config` for our cloud provider, here is GCE:

```sh
PROJECT_ID=$(curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/project/project-id)

cat <<EOF | sudo tee /etc/kubernetes/cloud-config
[Global]
project-id = "${PROJECT_ID}"
EOF
```

Reload and restart:

```
{
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet
}
```

## Enable HTTP Health Checks

GCP's TCP Load Balancer is tricky whiling backend services offer only HTTPS health check endpoint with non-standard port (say `6443`).
Anyway, spin up `nginx` to make it happy, as the workaround -- kindly let me know this issue has been fully addressed.

```
{
  sudo apt-get install -y nginx

  cat <<EOF | sudo tee /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF

  sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/

  sudo systemctl restart nginx
  sudo systemctl enable nginx
}
```

Next: [04-Bootstrapping and Joining Controllers](04-init-join-controllers.md)
