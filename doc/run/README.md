# Run: let Argo CD select Components

You can also keep an overlay generic and select Components in the Argo CD `Application`. This is useful when component selection belongs with the app registration or platform configuration. It requires Argo CD v2.10.0 or later, and the component paths are relative to `spec.source.path`.

The example Application points at `../examples/apps/catalog/overlays/argocd` and opts into both Components in [`../examples/argocd/catalog-prod.yaml`](../examples/argocd/catalog-prod.yaml). Apply it to a cluster where Argo CD is installed:

```sh
kubectl apply -n argocd -f doc/examples/argocd/catalog-prod.yaml
argocd app get catalog-prod
argocd app diff catalog-prod
```

The Application's destination namespace is `catalog`. The repository URL is a placeholder: replace `https://github.com/OWNER/REPO.git` with this repository's URL before applying. The target revision is `main`.

`kubectl kustomize doc/examples/apps/catalog/overlays/argocd` renders the generic overlay alone; it cannot see the Application's `spec.source.kustomize.components`. For the exact Argo CD result, inspect the Application's rendered manifests or run `argocd app diff`. If you want local builds and Argo CD to share one obvious source of component selection, declare `components` in the overlay as the production overlay does.

## Further reading

- [Kustomize Components example](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/components.md)
- [Argo CD: Kustomize](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)
- [Kubernetes: Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
