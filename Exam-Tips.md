# Exam Tips

## Familiarize Yourself with Kubernetes.IO Documentation

Always use Ctrl+F on the page. Don't scroll.

## Autocomplete

[More Info](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

```sh
alias k=kubectl

source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

## Use 'kubectl help' for examples

kubectl help provides good examples of how to complete a command.

## Alias Common Commands

```sh
alias k=kubectl
complete -o default -F __start_kubectl k

# short alias to set/show context/namespace (only works for bash and bash-compatible shells, current context to be set before using kn to set namespace)
alias kx='f() { [ "$1" ] && kubectl config use-context $1 || kubectl config current-context ; } ; f'
alias kn='f() { [ "$1" ] && kubectl config set-context --current --namespace $1 || kubectl config view --minify | grep namespace | cut -d" " -f6 ; } ; f'
```

## Use 'Explain' to See Resource Definitions

```sh
kubectl explain pods
```

## Temporary Pods for Troubleshooting

[Example](https://blog.derlin.ch/kubectl-run-spawn-temporary-docker-containers-on-kubernetes)

```sh
# run bash in a container with psql installed 
# on your namespace
kubectl --namespace myproject \
  run -it --rm psql --image=postgres:13 -- bash;

# connect to the db
PGPASSWORD='my-user-pwd' psql postgres -U my-user \
  -h hostname-of-the-db-cluster.rds.amazonaws.com

# add labels in your run command to work around network policies
kubectl run ... -l "db-access: true" -l "role: alice" ...
```

## Bookmark Frequently Used Docs

[k8s.io](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

[k8s Upgrades](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

## Know Exam Layout

1. Passing Score
2. Testing Environment
3. Time Limit

## Browser Tabs

You can have multiple Tabs open.
Use k8s Cheat Sheets
Documentation

## Know "Abstract" Questions

Know that multiple questions do not include the tool to be used.

Adjust a container so that every 5 seconds a command is sent to check its alive.

This is a Liveness Probe.

## Review Exam Dumps

https://www.itexams.com/exam/CKA

https://github.com/fahmifahim/kubernetes-cka/blob/master/02.CKA_practice.md

