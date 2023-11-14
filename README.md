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

## Examples

### Dynamically Re-Balance Clusters Across Shards
The Argo CD application controller can become overloaded when managing multiple remote clusters with many resources. The solution is to run multiple shards of the application controller, where each is responsible for one or more clusters (and each cluster only belongs to one shard). However, adding new shards requires increasing the number of replicas on the `application-controller` `StatefulSet`. Previously, the shard count was set via the `ARGOCD_CONTROLLER_REPLICAS` environment variable, and changing it forced a restart of all `application-controller` pods. In 2.9, this **alpha feature** updates the `application-controller` to be a `Deployment` (instead of a `StatefulSet`). The `replicas` field is now used for the shard count, and when replicas are added or removed, the sharding algorithm is re-run to ensure that the clusters are distributed accordingly.

Thanks, Ishita Sequeira (Red Hat), for writing the initial proposal and [implementing the whole feature](https://github.com/argoproj/argo-cd/pull/15036)!



### Ignore ApplicationSet Differences
The ApplicationSet is a powerful feature that renders Argo CD Applications dynamically based on generators. However, there are situations where another process revises certain fields of an Application after the ApplicationSet creates it. A common example is when using the Argo CD Image Updater with Applications managed by an ApplicationSet. When using the imperative write-back method, Image Updater changes the image tag using a parameter override (argocd app set --parameter …), which, if not ignored, would be overwritten by the ApplicationSet. Or if you simply want to be able to disable auto-sync for one Application managed by an ApplicationSet temporarily.

This was one of the most voted proposals for this release! Thank you, Michael Crenshaw (Intuit), for all your interaction with the community and for making this feature happen!

### Support for Inline Kustomize Patches
Argo CD Applications now support in-line Kustomize patches. These will be merged with any existing patches in the source `kustomization.yaml`.