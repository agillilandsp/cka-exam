# Networking

## Switch Routing

Switch is connected to two devices. Each Device has an interface

ip link: eth0

ip addr add ... # to add an address

Router is a tool to connect two separate networks.

`route` is used to see the routing table.

Creating a Route from A to C.

![Switching and Routing](./RoutingSwitching.png "Switching and Routing")

```sh
ip route add 192.168.2/24 via 192.168.1.6 # Device B
```

Creating a Route from A to C.

```sh
# on device A
ip route add 192.168.2.0/24 via 192.168.1.6 # Device B
```

Creating a Route from C to A.

```sh
# on device A
ip route add 192.168.1.0/24 via 192.168.1.6 # Device B
```

To forward packets

```sh
echo 1 > /proc/sys/net/ipv4/ip_forward
# must set in network interfaces file to persist.
```

```sh
ip link # list interfaces on host 
ip addr # see addresses on interfaces 
ip addr add # add ip address to interface
ip route add # add a route
# must set in network interfaces file to persist.
```

## Prerequisites - DNS/CoreDNS

Prerequisite - CoreDNS
In the previous lecture we saw why you need a DNS server and how it can help manage name resolution in large environments with many hostnames and Ips and how you can configure your hosts to point to a DNS server. In this article we will see how to configure a host as a DNS server.

We are given a server dedicated as the DNS server, and a set of Ips to configure as entries in the server. There are many DNS server solutions out there, in this lecture we will focus on a particular one – CoreDNS.

So how do you get core dns? CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.

![DNS](./dns.png "dns")

Run the executable to start a DNS server. It by default listens on port 53, which is the default port for a DNS server.

Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers /etc/hosts file.

And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.

![DNS](./dns2.png "dns")

CoreDNS also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later section.

Read more about CoreDNS here:

[CoreDNS Specifications](https://github.com/kubernetes/dns/blob/master/docs/specification.md)

[CoreDNS Plugin](https://coredns.io/plugins/kubernetes/)

## Prerequisite - Network Namespaces

If a Network is your house, the Namespaces are the rooms.

```sh
ip net add netns

ip netns exec red(ns) iplink

ip -n red link

ip link add veth-red type veth peer name veth-blue

ip link set veth-red netns red

ip link set veth-blue netns blue
```

Linux Bridge and Open VSwitch (OVS) are two virtual bridge networks on Linux

![alt text](./LinuxBridge.png "Linux Bridge")

## Prerequisites - Docker Networking

docker0 = bridge (inside docker)

Docker uses iptables directly on the host to forward ports.

## Prerequisites - CNI

Nearly every implementations was going through the same commands to achieve the same networking isolation, forwarding, security goals.

CNIs Responsibility

- Container Runtime must create network namespace
- Identify Network the container must attach to
- Conatiner Runtime to invoke Network Plugin (bridge) when container is ADDed
- Container Runtime to invoke... DELeted
- JSON Format of the Network Configuration

- Must support ADD/DEL/CHECK cli
- Must support parameters container id, network ns, etc...
- Must manage IP Address assignment to PODs
- Must Return results in a specific format

Docker does not implement CNI but Container Network Model. So when Docker is used on K8s, K8s deploys the container with no network and manually adds a bridge.

## Cluster Networking

Nodes must have IPs, MACs and FQDN names.

Ports that are frequently used:

- 6443 (kube-api) Masters
- 10250 (kubelet) Masters and Workers
- 10259 (scheduler) Masters
- 10257 (contoller-manager) Masters
- 2379 (etcd) Masters
- 2380 (etcd clients) Masters

![alt text](./required-ports.png)

[Required Ports](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)

## Important Note

In the upcoming labs, we will work with Network Addons. This includes installing a network plugin in the cluster. While we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

[K8s Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

[K8s Admin Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)

In the CKA exam, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.

However, the documentation currently does not contain a direct reference to the exact command to be used to deploy a third-party network addon.

The links above redirect to third-party/vendor sites or GitHub repositories, which cannot be used in the exam. This has been intentionally done to keep the content in the Kubernetes documentation vendor-neutral.

NOTE: In the official exam, all essential CNI deployment details will be provided.

Lab Notes...

Question 9. Finding your default gateway IP...

```sh
controlplane ~ ✖ ip route |grep default
default via 169.254.1.1 dev eth0 
```

Question 10. Finding kube-scheduler port...

```sh
controlplane ~ ✖ netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3516/kube-scheduler 
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      3841/kube-controlle 
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      4100/kubelet        
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      4583/kube-proxy     
tcp        0      0 127.0.0.1:34635         0.0.0.0:*               LISTEN      950/containerd      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      952/sshd: /usr/sbin 
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      3181/etcd           
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      3181/etcd           
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      954/ttyd            
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      530/systemd-resolve 
tcp        0      0 192.168.183.250:2379    0.0.0.0:*               LISTEN      3181/etcd           
tcp        0      0 192.168.183.250:2380    0.0.0.0:*               LISTEN      3181/etcd           
tcp6       0      0 :::10256                :::*                    LISTEN      4583/kube-proxy     
tcp6       0      0 :::10250                :::*                    LISTEN      4100/kubelet        
tcp6       0      0 :::8888                 :::*                    LISTEN      4282/kubectl        
tcp6       0      0 :::22                   :::*                    LISTEN      952/sshd: /usr/sbin 
tcp6       0      0 :::6443                 :::*                    LISTEN      3559/kube-apiserver 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           530/systemd-resolve 
udp        0      0 0.0.0.0:8472            0.0.0.0:*                           -                   
```

Question 11. Number of connections on etcd ports (which has more)

```sh
controlplane ~ ➜  netstat -anp | grep ":2380"| wc -l
1

controlplane ~ ➜  netstat -anp | grep ":2379"| wc -l
120
```

## Pod Networking

![alt text](./pod-networking1.png)

```sh
# Then add

# Bring Up Interface
ip -n <namespace> link set
```

The CNI uses a set of commands implemented at a particular address of the host machine...

![alt text](./cni-net-script.png)

## CNI Weave

Lab Notes...

Question 1.

ANSWER

```sh
ps -aux | grep -i kubelet | grep container-runtime
```

Question 2.

```sh
# path to all the binaries of CNI supported plugins
ls -la /opt/cni/bin/
```

What plugins are available?

```sh
ls -la /opt/cni/bin/
```

What is the Configured Plugin?

```sh
ls -la /etc/cni/net.d/
```

What binary executable file will be run by kubelet after a container and its associated namespace are created?

SOLUTION:

```sh
cat /etc/cni/net.d/10-flannel.conflist
```

## Practice Test Deploy Network Solution

Lab Notes...

Pod behavior when there is no network configured.

Here weave-net was not deployed.

```sh
Events:
  Type     Reason                  Age                From               Message
  ----     ------                  ----               ----               -------
  Normal   Scheduled               111s               default-scheduler  Successfully assigned default/app to controlplane
  Warning  FailedCreatePodSandBox  110s               kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "2450aa405e8b5ef132181632744704fdfb423252a066ca45d964faa2ba87faf7": plugin type="weave-net" name="weave" failed (add): unable to allocate IP address: Post "http://127.0.0.1:6784/ip/2450aa405e8b5ef132181632744704fdfb423252a066ca45d964faa2ba87faf7": dial tcp 127.0.0.1:6784: connect: connection refused
  Normal   SandboxChanged          2s (x9 over 109s)  kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   Pulling                 1s                 kubelet            Pulling image "busybox"
  Normal   Pulled                  1s                 kubelet            Successfully pulled image "busybox" in 334ms (334ms including waiting). Image size: 2167089 bytes.
  Normal   Created                 1s                 kubelet            Created container app
```

Deploy Weave Net by applying the manifest.

```yaml
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: weave-net
      labels:
        name: weave-net
      namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: weave-net
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - pods
          - namespaces
          - nodes
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - extensions
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - 'networking.k8s.io'
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
        - ''
        resources:
        - nodes/status
        verbs:
        - patch
        - update
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      labels:
        name: weave-net
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: weave-net
      namespace: kube-system
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - configmaps
        resourceNames:
          - weave-net
        verbs:
          - get
          - update
      - apiGroups:
          - ''
        resources:
          - configmaps
        verbs:
          - create
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: weave-net
      namespace: kube-system
      labels:
        name: weave-net
    roleRef:
      kind: Role
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: weave-net
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      # Wait 5 seconds to let pod connect before rolling next pod
      selector:
        matchLabels:
          name: weave-net
      minReadySeconds: 5
      template:
        metadata:
          labels:
            name: weave-net
        spec:
          initContainers:
            - name: weave-init
              image: 'weaveworks/weave-kube:2.8.1'
              command:
                - /home/weave/init.sh
              env:
              securityContext:
                privileged: true
              volumeMounts:
                - name: cni-bin
                  mountPath: /host/opt
                - name: cni-bin2
                  mountPath: /host/home
                - name: cni-conf
                  mountPath: /host/etc
                - name: lib-modules
                  mountPath: /lib/modules
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
              env:
                - name: IPALLOC_RANGE
                  value: 10.32.1.0/24
                - name: INIT_CONTAINER
                  value: "true"
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'weaveworks/weave-kube:2.8.1'
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
              resources:
                requests:
                  cpu: 50m
              securityContext:
                privileged: true
              volumeMounts:
                - name: weavedb
                  mountPath: /weavedb
                - name: dbus
                  mountPath: /host/var/lib/dbus
                  readOnly: true
                - mountPath: /host/etc/machine-id
                  name: cni-machine-id
                  readOnly: true
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
            - name: weave-npc
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'weaveworks/weave-npc:2.8.1'
#npc-args
              resources:
                requests:
                  cpu: 50m
              securityContext:
                privileged: true
              volumeMounts:
                - name: xtables-lock
                  mountPath: /run/xtables.lock
                  readOnly: false
          hostNetwork: true
          dnsPolicy: ClusterFirstWithHostNet
          hostPID: false
          restartPolicy: Always
          securityContext:
            seLinuxOptions: {}
          serviceAccountName: weave-net
          tolerations:
            - effect: NoSchedule
              operator: Exists
            - effect: NoExecute
              operator: Exists
          volumes:
            - name: weavedb
              hostPath:
                path: /var/lib/weave
            - name: cni-bin
              hostPath:
                path: /opt
            - name: cni-bin2
              hostPath:
                path: /home
            - name: cni-conf
              hostPath:
                path: /etc
            - name: cni-machine-id
              hostPath:
                path: /etc/machine-id
            - name: dbus
              hostPath:
                path: /var/lib/dbus
            - name: lib-modules
              hostPath:
                path: /lib/modules
            - name: xtables-lock
              hostPath:
                path: /run/xtables.lock
                type: FileOrCreate
          priorityClassName: system-node-critical
      updateStrategy:
        type: RollingUpdate
```

Solution covers more regarding weavenet deployments and configuring from the online manifest. The solution covers the use of the config map in the kube-proxy and using that for the IP_ALLOC field of the weavenet manifest.

[Follow up](https://samaritanspurse.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/21739724#overview)

## IP Address Management Weave

```sh
cat /etc/cni/net.d/net-script.conf
...
"ipam": {
  "type": "host-local",
  "subnet": ...,
  "routes: [
    {"dst":...}
    ]
  }
}
```

Lab Notes...

Question 6 What is the IP Range in use.

Describe weave pods or daemonset and find the `IPALLOC_RANGE:`

Question 7 What is the default gateway on the node01?

Deploy a pod (use nodeSelector or nodeName)

```sh
kubectl run nginx --image=nginx

k get po -A -owide

node01 ~ ➜  ip route
default via 172.25.0.1 dev eth1 
10.244.0.0/16 dev weave proto kernel scope link src 10.244.192.0 
172.25.0.0/24 dev eth1 proto kernel scope link src 172.25.0.70 
192.19.4.0/24 dev eth0 proto kernel scope link src 192.19.4.3 
```

SOLUTION: Deploying a pod on node01

```sh
kubectl run busybox --image=busybox --dry-run -o yaml --sleep 1000 > busybox.yaml

# Add node name under spec...

...
spec:
  nodeName: node01
  containers:
...
```

```sh
k exec busybox -- ip route
default via 10.244.192.0 dev eth0
10.244.0.0/16 dev eth0 scope link src 10.244.192.1
```

## Service Networking

ClusterIP... available only to the cluster.

NodePort... same as ClusterIP but has external access to one ip on one node.

Services in detail...

Each kube-proxy gets the allocated ip for the svc from k8s-apiserver and creates forwarding rules.

Different methods for forwarding...

- iptables (default)
- ipvs
- userspace

```sh
# startup kube-api-server --service-cluster-ip-range ipNet (Default 10.0.0.0/24)
cat /etc/kubernetes/manifests/kube-apiserver.yaml|grep service-cluster-ip
```

Do not overlap your networks (svcs and pods)

Inspect the forwarding rules using the iptables...

```sh
k get service
NAME...
db-service...

iptables -L -t nat | grep db-service

# check log (you need the correct verbosity set)
cat /var/log/kube-proxy.log
```

Lab notes...

```sh
# Find which ip forwarding/proxy is being used by kube-proxy.

controlplane ~ ✖ k logs kube-proxy-pf4gp -n kube-system
I1217 22:13:51.393085       1 server_linux.go:66] "Using iptables proxy"
I1217 22:13:51.568328       1 server.go:677] "Successfully retrieved node IP(s)" IPs=["192.19.102.3"]
I1217 22:13:51.590697       1 conntrack.go:60] "Setting nf_conntrack_max" nfConntrackMax=1179648
I1217 22:13:51.592094       1 conntrack.go:121] "Set sysctl" entry="net/netfilter/nf_conntrack_tcp_timeout_established" value=86400
E1217 22:13:51.593323       1 server.go:234] "Kube-proxy configuration may be incomplete or incorrect" err="nodePortAddresses is unset; NodePort connections will be accepted on all local IPs. Consider using `--nodeport-addresses primary`"
I1217 22:13:51.612607       1 server.go:243] "kube-proxy running in dual-stack mode" primary ipFamily="IPv4"
I1217 22:13:51.612665       1 server_linux.go:169] "Using iptables Proxier"
# NOTE THIS LINE ABOVE.
```

```sh
# Lab History
k get nodes -o wide
ip a
k get po -o wide
k get po -A -o wide
k logs weave-net-bvmx7
k logs weave-net-bvmx7 -n kube-system
k get svc -o wide
k get svc -o wide -A
cat /etc/kubernetes/manifests/kube-apiserver.yaml |grep ip-range
k get po -A
k logs kube-proxy-pf4gp
k logs kube-proxy-pf4gp -n kube-system
k get svc
k get svc -A
iptables -L -t nat |grep kube-dns
k get ds
k get ds -A
```

## DNS in Kubernetes

Pods use dashes on their IP.

service.namespace.svc.cluster.local

## CoreDNS K8s

```sh
cat /etc/resolv.conf

cat /etc/hosts

cat /etc/coredns/Corefile

kubelet updates the kube-dns
```

Lab Notes...

Question 5. Where is the Corefile?

Mounted as a config map at /etc/coredns

Inspect the pods and get the config map information.

Inspect the config map for coredns.

```sh
k -n kube-system  get po
k -n kube-system describe po coredns-77d6fd4654-x4fht
k -n kube-system get po coredns-77d6fd4654-x4fht -o yaml
k get cm -n kube-system
k get cm dns -n kube-system
k get cm coredns -n kube-system
k get cm coredns -n kube-system -o yaml
k get po -A
k get po,svc -A
k exec test -- curl http://web-service.payroll
k exec test -- curl http://hr
k get po -A -owide
k exec test -- curl http://172-17-0-5.hr.pod
k exec test -- curl http://172-17-0-5.hr
k get svc -A
k describe svc test-service
k describe svc web-service
k get po
k get deployment
k edit deployment webapp
k get op -A
k get po -A
k edit deployment webapp
k exec hr -- nslookup mysql
k exec hr -- nslookup mysql > /root/CKA/nslookup.out
cat /root/CKA/nslookup.out 
k get svc -A
k exec hr -- nslookup mysql.svc > /root/CKA/nslookup.out
cat /root/CKA/nslookup.out 
k exec hr -- nslookup mysql.payroll.svc > /root/CKA/nslookup.out
cat /root/CKA/nslookup.out 
```

## Ingress

NodePort...

* can only allocate ports above 30000

LoadBalancer...

* same as a standard NodePort
* has an external IP
* request for a LB allocated by the Cloud Provider

Ingress is a layer 7 load balancer integrated into the k8s cluster.

* Haproxy
* Traefik
* Nginx

Ingress Controllers (LB, SSL, Routing)

GCE GCP, NGINX

* Loadbalancer is just a part of it
* Monitors the cluster for changes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
    template:
      metadata:
        labels:
          name: nginx-ingress
      spec:
        containers:
          - name: nginx-ingress-controller
            image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
        
        args:
          - /nginx-ingress-controller
          - --configmap=$(POD_NAMESPACE)/nginx-configuration
            # Use a config map to inject here
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace

        ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
----
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

## Article: Ingress

As we already discussed Ingress in our previous lecture. Here is an update.

In this article, we will see what changes have been made in previous and current versions in Ingress.

Like in apiVersion, serviceName and servicePort etc.

![K8s-Ingress](ingress.png)
Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

Format - `kubectl create ingress <ingress-name> --rule="host/path=service:port"`

Example - `kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"`

Find more information and examples in the below reference link:-

[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-)

References:-

[https://kubernetes.io/docs/concepts/services-networking/ingress](https://kubernetes.io/docs/concepts/services-networking/ingress)

[https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)

## Ingress - Annotations and rewrite-target

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

Our watch app displays the video streaming webpage at http://<watch-service>:<port>/

Our wear app displays the apparel webpage at http://<wear-service>:<port>/

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/

Without the rewrite-target option, this is what would happen:

http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear

Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

For example: replace(path, rewrite-target)
In our case: replace("/path","/")

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```

Lab notes...

```yaml
# deployment/ingress-nginx-controller
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  generation: 1
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --watch-ingress-without-class=true
        - --default-backend-service=app-space/default-backend-service
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: registry.k8s.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: ingress-nginx
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          defaultMode: 420
          secretName: ingress-nginx-admission

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  clusterIP: 172.20.103.199
  clusterIPs:
  - 172.20.103.199
  externalTrafficPolicy: Local
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    nodePort: 32103
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  sessionAffinity: None
  type: NodePort

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2024-12-19T23:20:25Z"
  generation: 1
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 172.20.103.19

```

Question 8. What is the Host configured in the Ingress Resource... All (*)

SOLUTION (hosts is shown)

```sh
k get ingress -A
```

Question 11. default-backend-servce where is this configured?

SOLUTION: USE DESCRIBE (Default backend is shown)

ALWAYS DOUBLE CHECK WHAT RESOURCE THEY ARE ASKING YOU TO LOOK FOR!

```yaml
# pay ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-pay
  namespace: critical-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: pay-service
            port:
              number: 8282
        path: /pay
        pathType: Prefix
```

Declarative

```sh
k create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```

Lab 2 Ingress

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --watch-ingress-without-class=true
        - --default-backend-service=app-space/default-http-backend
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: registry.k8s.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort