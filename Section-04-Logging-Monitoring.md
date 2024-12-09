# Logging and Monitoring

Metrics Server

Only an in-memory analysis

Runs an agent on each node with cAdvisor

```bash
kubectl top node
```

Deploy metrics server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

kubectl logs... qualify the specific container you wish to grab the log for when there are multiple containers in a Pod.

```bash
kubectl logs <pod> -f [optional when more than one container in the pod <container-name>]
```
