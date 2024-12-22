# Deployment with Kubeadm

Notes...

> ON EACH NODE

Editing the `/etc/containerd/config.toml` file to include the systemdCgroup (this should be enabled as default)

```sh
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
```

`kubeadm init` preflight caught an issue with the ipv4.ip_forward setting.

Added this to fix it.

```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

sysctl net.ipv4.ip_forward
```

> USE the k8s blog for searching for these solutions.

Control Plane Node Kubadm Init Output:

```sh

vagrant@controlplane:~$ sudo kubeadm init --apiserver-advertise-address 192.168.0.40 --pod-network-cidr "10.200.0.0/16" --upload-certs
I1220 23:28:10.327221   35590 version.go:261] remote version is much newer: v1.32.0; falling back to: stable-1.31
[init] Using Kubernetes version: v1.31.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W1220 23:28:11.055115   35590 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.0.40]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controlplane localhost] and IPs [192.168.0.40 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controlplane localhost] and IPs [192.168.0.40 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 500.665782ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 5.001426935s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
c4f35a5898501e31e47356f189d049a5c7032c19728fc26d7233ce3c3ad9d82f
[mark-control-plane] Marking the node controlplane as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node controlplane as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 13eu4g.9prh2xjdxo3q2ahm
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addonwhat does modprobe do cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.40:6443 --token 13eu4g.9prh2xjdxo3q2ahm \
	--discovery-token-ca-cert-hash sha256:6f53545520620dcc6b28d5baf9a1136788eca2fbc42e9777ba873714e2099e5c 

```

```sh
# Flannel pod in CrashLoopBackoff
vagrant@controlplane:~$ k logs kube-flannel-ds-vj4dx -n kube-flannel
k: command not found
vagrant@controlplane:~$ kubectl logs kube-flannel-ds-vj4dx -n kube-flannel
Defaulted container "kube-flannel" out of: kube-flannel, install-cni-plugin (init), install-cni (init)
I1221 00:04:10.578681       1 main.go:211] CLI flags config: {etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379 etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile: etcdUsername: etcdPassword: version:false kubeSubnetMgr:true kubeApiUrl: kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:[] ifaceRegex:[] ipMasq:true ifaceCanReach: subnetFile:/run/flannel/subnet.env publicIP: publicIPv6: subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0 iptablesResyncSeconds:5 iptablesForwardRules:true netConfPath:/etc/kube-flannel/net-conf.json setNodeNetworkUnavailable:true}
W1221 00:04:10.578881       1 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I1221 00:04:10.584694       1 kube.go:139] Waiting 10m0s for node controller to sync
I1221 00:04:10.584852       1 kube.go:469] Starting kube subnet manager
I1221 00:04:11.585076       1 kube.go:146] Node controller sync successful
I1221 00:04:11.585212       1 main.go:231] Created subnet manager: Kubernetes Subnet Manager - controlplane
I1221 00:04:11.585232       1 main.go:234] Installing signal handlers
I1221 00:04:11.585576       1 main.go:468] Found network config - Backend type: vxlan
E1221 00:04:11.585775       1 main.go:268] Failed to check br_netfilter: stat /proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory
```

Suggested solution for error in Flannel Pod "Failed to check br_netfilter: stat /proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory"

```sh
sudo modprobe br_netfilter
sudo echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

Join the other nodes to the cluster:

```sh
sudo kubeadm join 192.168.0.40:6443 --token 13eu4g.9prh2xjdxo3q2ahm \
	--discovery-token-ca-cert-hash sha256:6f53545520620dcc6b28d5baf9a1136788eca2fbc42e9777ba873714e2099e5c
```

## Key Steps

1. Configure your CRI. You will need to select a runtime and configure the appropriate settings. Containerd has a default config it can generate once installed that then is edited to match `systemd` as your Cgroup.
2. Ensure you have ipv4 forwarding enabled. That is a simple file that contains the '1' at `/etc/sysctl.d/k8s.conf`. Also, check your `/proc/sys/net/ipv4/ip_forward` and `/proc/sys/net/bridge/bridge-nf-call-iptables` for the value of '1' in each.
3. Initialize the cluster from the control plane and pass in the arguements for `--upload-certs`, `--apiserver-advertise-address` \<interface ip of your controlplane\> --pod-network-cidr <defaults to 10.244.0.0/16 but you can set this>
4. If you chose to set your own pod CIDR, you must add that to your CNI network plugin.
5. If you have a multi-controlplane cluster you must add some arguments and setup a load balancer.
