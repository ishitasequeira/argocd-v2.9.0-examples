# Argo CD v2.9.0 Examples

## Bootstrapping Argo CD
```
kind create cluster --config kind-cluster.yaml
kubectl apply -k argocd
```

## Accessing the UI
Navigate to [https://localhost:8080/](https://localhost:8080/) on the machine with the `kind` cluster running.

Get the generated `admin` password.
```
argocd admin initial-password -n argocd
```

## Clean Up
```
kind delete cluster --name argocd-v2-9-0-examples
```