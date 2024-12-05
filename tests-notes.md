* KodeKloud Test Notes

** Pods Test

https://learn.kodekloud.com/user/courses/udemy-labs-certified-kubernetes-administrator-with-practice-tests/module/e6ae2f68-9b3a-439e-a534-d63d372840d2/lesson/5c388f3e-bd27-41bd-9c3c-613dafc29bf9

Using "Dry Run" to get YAML content for the "kubectl apply" command:

```sh
kubectl run redis --image=redis123 --dry-run=client -o yaml


kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```
