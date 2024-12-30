# Additional Study

Areas to focus:

* Network Policies [link](#network-policies)
* ServiceAccounts, User Accounts, Roles, RoleBindings [link](#serviceaccounts-user-accounts-roles-rolebindings)
* API Server Crashed [link](#api-server-crashed)
* Services (LB vs ClusterIP vs NodePort) [link](#services-lb-vs-clusterip-vs-nodeport)
  * Know Ingress
* OpenSSL and Cert Maintenance [link](#openssl-and-cert-maintenance)
* Adding a Node to a Cluster
* Kubeadm Install, Upgrade [link](#kubeadm-install-upgrade)
  * Specifically `apt` commands
* Misc Review [link](#misc-review)

## Network Policies

### KILLER CODA: NetworkPolicy Namespace Selector

<https://killercoda.com/killer-shell-cka/scenario/networkpolicy-namespace-communication>

There are existing Pods in Namespace space1 and space2 .

We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 . Incoming traffic not affected.

We also need a new NetworkPolicy named np that restricts all Pods in Namespace space2 to only have incoming traffic from Pods in Namespace space1 . Outgoing traffic not affected.

The NetworkPolicies should still allow outgoing DNS traffic on port 53 TCP and UDP.

```yaml
#np-1.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space2
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53

#np-2.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space1
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

```sh
# Verify...
# these should work
k -n space1 exec app1-0 -- curl -m 1 microservice1.space2.svc.cluster.local
k -n space1 exec app1-0 -- curl -m 1 microservice2.space2.svc.cluster.local
k -n space1 exec app1-0 -- nslookup tester.default.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 app1.space1.svc.cluster.local

# these should not work
k -n space1 exec app1-0 -- curl -m 1 tester.default.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 microservice1.space2.svc.cluster.local
k -n kube-system exec -it validate-checker-pod -- curl -m 1 microservice2.space2.svc.cluster.local
k -n default run nginx --image=nginx:1.21.5-alpine --restart=Never -i --rm  -- curl -m 1 microservice1.space2.svc.cluster.local
```

## ServiceAccounts, User Accounts, Roles, RoleBindings

> Service Accounts can only be accessed with `can-i` using the prefix `system:serviceaccount:<namespace>:<service account name>`

```sh
openssl genrsa -out test.key 2048

openssl req -new -key test.key -subj="/CN=test" -out test.csr 

cat test.csr | base64 | tr -d "/n" # This is copied into the CSR yaml

```

```yaml
# CSR Yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: test
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: <copied from cat output>
  usages:
  - digital signature #optional
  - key encipherment #optional
  - client auth
```

Then approve the csr...

```sh
kubectl certificate approve test
```

Assign it to roles...

```sh
k create rolebinding binding-test --user=test --clusterrole=test-role
```

Verify Permissions

```sh
kubectl auth can-i <verb> <resource> --as=test --namespace=<namespace>
```

OPTIONAL: How to connect on kubectl using that new usr/role

```sh
kubectl get csr test -o jsonpath='{.status.certificate}'| base64 -d > test.crt
```

Embed the certificate to set a new context on kubectl config.

```sh
kubectl config set-credentials ajg-ops --client-key=test.key --client-certificate=test.crt --embed-certs=true
```

Create a context and use it.

```sh
kubectl config set-context test@clustername --cluster=<clustername> --user=test
```

## API Server Crashed

### KILLER CODA: API Server Crashed

The idea here is to misconfigure the Apiserver in different ways, then check possible log locations for errors.

You should be very comfortable with situations where the Apiserver is not coming back up.

Configure the Apiserver manifest with a new argument --this-is-very-wrong .

Check if the Pod comes back up and what logs this causes.

Fix the Apiserver again.

Know the log locations: 
`/var/log/pods`
`/var/log/containers`

Know how to inspect pods/containers while apiserver is down:
`crictl ps` + `crictl log`
`docker ps` + `docker logs` (in case when Docker is used)

Know where kubelet logs are:
`kubelet logs: /var/log/syslog or journalctl`

### KILLER CODA: NetworkPolicy Misconfigured

<https://killercoda.com/killer-shell-cka/scenario/networkpolicy-misconfigured>

## Services (LB vs ClusterIP vs NodePort)

[Ingress Review](./Section-09-Networking.md#article-ingress)

## OpenSSL and Cert Maintenance

[Certs Review](./Section-16-Mock-Exams.md#q6)

Check Certificate Expiration

```sh
openssl x509 -in cert.crt -text -noout

# output
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            30:6a:bf:fa:0f:5b:1f:83:91:92:31:3e:04:0a:e0:8b
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Dec 29 15:58:54 2024 GMT
            Not After : Dec 29 15:58:54 2025 GMT
        Subject: CN = ajg-ops
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:ae:40:39:19:65:12:24:e2:3d:8d:96:fc:3f:b0:
                    4b:93:bd:c1:8a:83:b6:48:13:70:a4:bb:b4:a5:d5:
                    d7:e2:18:7b:fb:95:70:cc:7e:2f:df:8c:c4:5c:6b:
                    19:21:ea:55:6d:be:21:0f:32:bb:b3:db:c2:cb:7c:
                    f9:40:d4:49:5c:47:b8:cf:1d:b5:9d:c1:11:57:c3:
                    0f:51:b2:62:a2:13:ed:39:86:51:c3:fe:52:43:46:
                    54:b2:fa:e4:13:9d:c8:74:95:48:57:91:63:4f:c1:
                    1a:da:88:65:17:5e:81:8b:3a:31:86:bb:cd:86:8d:
                    01:5f:48:09:97:65:75:2e:e7:a6:cd:c8:22:c8:62:
                    86:28:33:4a:a3:80:52:d9:3b:31:ff:e3:5e:a8:98:
                    1e:94:f0:a9:a7:cb:b8:08:2a:d1:1e:bb:e0:00:38:
                    f2:dd:66:6f:9f:2a:53:3e:18:02:9d:41:c1:51:55:
                    e7:bf:db:ef:05:11:6c:ea:ee:6f:72:17:84:bb:4d:
                    2e:d8:f5:b7:d9:a5:65:ed:e2:98:98:10:5d:d8:87:
                    bb:16:26:77:70:e0:5d:33:0b:f3:c4:27:db:be:3a:
                    fc:5f:70:3c:53:32:76:39:cf:cd:25:d2:4e:8e:98:
                    1d:15:c5:b6:a5:e3:99:6d:f1:3c:fd:df:84:e3:eb:
                    33:ad
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                9D:25:59:71:44:2E:73:48:9C:93:0F:52:98:D9:C6:06:A8:AF:42:71
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        7e:e7:06:d8:38:79:ec:67:46:97:ab:db:60:52:7d:d3:3e:e7:
        96:6f:68:f4:af:b0:d9:36:fe:f7:4f:69:84:3f:ac:c6:f6:f2:
        de:67:c7:89:5c:bc:cf:85:65:8a:70:6c:59:3e:98:12:8e:d0:
        65:42:1f:8f:b7:c3:2a:9e:df:04:a8:b0:bd:53:dc:ba:a8:7b:
        b1:e2:b2:62:57:0d:0e:d0:b3:33:b2:62:4e:ba:9c:86:ac:94:
        e2:ca:78:1d:4e:5f:ae:36:8c:34:f0:0f:e5:68:8b:df:62:38:
        e8:ae:3b:b6:04:26:c8:ad:18:46:03:21:f2:09:42:ae:7a:5b:
        3a:1b:bc:35:c7:94:9b:0f:81:c2:e5:e3:02:e0:4f:94:83:91:
        ad:98:a3:c2:ca:e2:fb:83:36:0e:60:20:18:cc:5b:3e:5d:9a:
        cb:40:15:70:ed:38:90:04:f1:ce:71:42:18:20:18:01:86:2b:
        33:bb:27:5b:35:65:a5:d6:cd:8c:df:3d:ab:1f:af:c1:65:4d:
        aa:f3:e5:df:ff:7e:fd:3f:2d:61:88:5c:d8:c2:19:7a:1b:21:
        b5:cd:0b:b6:af:3b:ac:99:f5:0e:69:8d:64:b3:49:9b:98:9a:
        b6:2e:3a:a1:7e:2b:2d:80:24:9c:31:15:79:3e:d1:ae:ce:2f:
        36:46:29:5e
```

Pay attention to Issuer and other important info. Use grep accordingly.

Check and renew certificates on the k8s cluster using kubeadm.

```sh
kubeadm certs check-expiration

kubeadm certs renew all

kubectl renew apiserver
```

## Adding a Node to a Cluster

[Join a Node to a Cluster](./Section-100-KillerShellExams.md#q20)

## Kubeadm Install, Upgrade

[Cluster Upgrade](./Section-06-ClusterMaintenance.md#cluster-upgrade-commands)

## Misc Review

Marked for Study:

[Control Plane Node Failure](./Section-13-Troubleshooting.md#control-plane-failure)

[Worker Node Failure](./Section-13-Troubleshooting.md#worker-node-failure)

[Image Security Review](./Section-07-Security.md#image-security)

[Resolving Names in Cluster](./Section-16-Mock-Exams.md#q7)

[Configuration/Installing CNI (Weave)](./Section-09-Networking.md#practice-test-deploy-network-solution)

olution covers more regarding weavenet deployments and configuring from the online manifest. The solution covers the use of the config map in the kube-proxy and using that for the IP_ALLOC field of the weavenet manifest.

> NOTE: Check the configmap or kube-proxy (not the local file system) to find the arguments on --cidr needed.

<https://samaritanspurse.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/21739724#overview)>

[CNI Weave](./Section-09-Networking.md#cni-weave)