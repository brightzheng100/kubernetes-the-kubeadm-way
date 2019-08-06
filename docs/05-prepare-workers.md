# Preparing Workers

## SSH Into Workers

You're encouraged to use multi-pane env, like [tmux](https://github.com/tmux/tmux/wiki) or [iTerm2](https://www.iterm2.com/), so you can run below steps in parallel for each Controller.
- In pane 1, SSH into `k8s-worker-0`: `gcloud compute ssh k8s-worker-0`
- In pane 2, SSH into `k8s-worker-1`: `gcloud compute ssh k8s-worker-1`
- In pane 3, SSH into `k8s-worker-2`: `gcloud compute ssh k8s-worker-2`


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
cat <<EOF | sudo tee vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
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

Reload and restart:

```
{
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet
}
```

Next: [06-Joining Workers](06-join-workers.md)
