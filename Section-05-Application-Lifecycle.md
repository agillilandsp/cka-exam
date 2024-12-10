# Section 5 Application Lifecycle

Rolling Updates/Rollbacks

Recreate takes down each pod on each node then starts deploying the new versions.

Rolling Update is the assumed default strategy.

A new Deployment creates a new rollout. That creates a Revision 1. Subsequent deployments are Revision 2...

Rollout Status:

```bash
kubectl rollout status deployment/myapp-deployment
```

Rollout History:

```bash
kubectl rollout history deployment/myapp-deployment
```

Upgrades:

Creates a new ReplicaSet

```bash
kubectl rollout undo deployment/myapp-deployment
```

Summary

```bash
# Create
kubectl create -f deployment.yaml

# Get
kubectl get deployemnts

# Update
kubectl apply -f deployment-update.yaml
#OR
kubectl set image deployment/myapp.yaml

# Status
kubectl rollout status deployment/myapp

# History
kubectl rollout history deployment/myapp

#Rollback
kubectl rollout undo deployment/myapp
```

## Commands

Some containers do not have anything but the shell `bash` present.

```dockerfile
FROM ubuntu

CMD sleep 5

CMD ["sleep","5"]

# can be overwritten by docker run ubuntu sleep 10
```

```dockerfile

from 

ENTRYPOINT ["sleep"]

```

## Commands and Arguments

```yaml
apiVersion: v1
kind: Pod
...
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

Equals...

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Note the `args` overrides the `CMD` and the `command` overrides the `ENTRYPOINT`.

Lab notes...

```bash
# Overriding CMD (args)
kubectl run nginx --image=nginx -- <arg1> <arg2> ...
# Overrideing ENTRYPOINT (command)
kubectl run nginx --image --command <cmd> <arg>...
```

> BE CAREFUL! Do not use arguments in command because it will override entrypoint and only pass your flags to the container.

## Configure Environment Variables in Applications

## Configuring ConfigMaps in Applications

```yaml
kubectl create configmap appconfig --from-literal=APP_COLOR=blue

... --from-file
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: x-config
...
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```yaml
# Inject it into a pod...
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: x
    image: x:v1
    ports:
      - containerPort: 8080
    envFrom: # Whole Config Map
      - configMapRef:
          name: x-config
    ...
    env: # One entry
      - name: APP_COLOR
        valueFrom: 
          configMapKeyRef:
            name: x-config
            key: APP_COLOR
    ...
    volumes:
    - names: x-config-volume
      configMap
        name: x-config

```

> NOTE: The instructor edits the pod and uses the generated /tmp/file...yaml by forcing it `kubectl replace --force`.

### Configure Secrets in Applications

```sh
kubectl create secret generic...

  app-secret --from-literal=DB_Host=mysql

  #OR...

  app-secret --from-file=./path/to/file.properties.
```

Secrets are injected like ConfigMaps above.

Secrets are not encrypted but only encoded.

> NOTE: Secret CSI Driver [link](https://www.youtube.com/watch?v=MTnQW9MxnRI)


## Encryption at Rest

ETD Data is not encrypted at rest.

You can see this by checking the ETCD DB.

```sh
ETCD_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret | hexdump -C
```

Add an encryption provider to your `/etc/kubernetes/manifest/kubeapi-server.yaml` file.

```yaml
# kubeapi-sever.yaml
--encrption-provider-config=/etc/kubernetes/enc/enc.yaml


# provider file enc.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets:
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: [CONTENTS From head -c 32 /dev/urandom | base64]

# Add these as volume mounts.

# Replace all previously made secrets.

kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

> NOTE: REVISIT THIS

## Multi-container Pods

Know how to create sidecar containers in pods. Especially useful for logging.

For Development there are two others that are helpful on multi-container design patters, ambassador and adapter.

## InitContainers

## Self Healing Applications

Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.
