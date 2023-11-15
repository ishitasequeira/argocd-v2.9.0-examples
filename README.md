# Argo CD v2.9.0 Examples

## Bootstrapping Argo CD
```
kind create cluster --config kind-cluster.yaml
kubectl apply -k argocd
kubectl apply -f apps.yaml
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
The Argo CD application controller can become overloaded when managing multiple remote clusters with many resources. The solution is to run multiple shards of the application controller, where each is responsible for one or more clusters (and each cluster can only belongs to one shard). However, adding new shards requires increasing the number of replicas on the `application-controller` `StatefulSet`. Previously, the shard count was set via the `ARGOCD_CONTROLLER_REPLICAS` environment variable, and changing it forced a restart of all `application-controller` pods. In 2.9, this **alpha feature** updates the `application-controller` to be a `Deployment` (instead of a `StatefulSet`). The `replicas` field is now used for the shard count, and when replicas are added or removed, the sharding algorithm is re-run to ensure that the clusters are distributed accordingly.

The `application-controller` `Deployment` relies on a config map to track the mapping of replica (pod) to shard number, and the heartbeat from each shard. If the `argocd-app-controller-shard-cm` does not exist in the same namespace as the `application-controller`, it will create it. Then a shard number (`ShardNumber`) is associated with a `Pod` name (the `ControllerName`). Each shard will update it's heartbeat time (`HeartbeatTime`) when the `readinessProbe` on the `Pod` is called, which is every 10 seconds be default.

```json
[{
    "ShardNumber":0,
    "ControllerName":"argocd-application-controller-58c9456b8d-6s5r2","HeartbeatTime":"2023-11-15T15:27:41Z"
},
{
    "ShardNumber":1,
    "ControllerName":"argocd-application-controller-58c9456b8d-n4vfq","HeartbeatTime":"2023-11-15T15:27:42Z"
}]
```

Users of the default hash-based sharding algorithm won't see any improvements as clusters be roughly-balanced between shards. The dynamic shard re-balancing functionality benefits users of custom shard assignment strategies the most. For users of the round-robin or other custom algorithms, a static assignment can lead to unbalanced shards when replicas are added or removed.

Thanks, Ishita Sequeira (Red Hat), for writing the initial proposal and [implementing the whole feature](https://github.com/argoproj/argo-cd/pull/15036)!

https://argo-cd.readthedocs.io/en/latest/operator-manual/dynamic-cluster-distribution/

### Ignore ApplicationSet Differences
The ApplicationSet is a powerful feature that renders Argo CD Applications dynamically based on generators. However, there are situations where another process revises certain fields of an Application after the ApplicationSet creates it. A common example is when using the Argo CD Image Updater with Applications managed by an ApplicationSet. When using the imperative write-back method, Image Updater changes the image tag using a parameter override (argocd app set --parameter …), which, if not ignored, would be overwritten by the ApplicationSet. Or if you simply want to be able to disable auto-sync for one Application managed by an ApplicationSet temporarily.

This was one of the most voted proposals for this release! Thank you, Michael Crenshaw (Intuit), for all your interaction with the community and for making this feature happen!

### Support for Inline Kustomize Patches
Argo CD Applications now support in-line Kustomize patches. These will be merged with any existing patches in the source `kustomization.yaml`.