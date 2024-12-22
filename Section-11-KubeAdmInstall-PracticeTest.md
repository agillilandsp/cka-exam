# Section 10 Kubernetes Install Practice Test

Install Kubeadm and Kubelet at specific versions.

```sh
sudo apt update
sudo apt-get install -y kubeadm='1.31.0-1.1'
sudo apt-get install -y kubelet='1.31.0-1.1'
```

> NOTE: I missed installing the containerd runtime and the cgroup drivers. Do that first.

Before `kubeadm`...

1. Setup ipv4 forwarding.
2. Install containerd
3. Configure containerd
