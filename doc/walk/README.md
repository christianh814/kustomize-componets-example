# Walk: add reusable Components

A **Component** is an opt-in Kustomize package for a capability that can extend or transform resources assembled by a base or overlay. Its `kustomization.yaml` uses `kind: Component`. The consumer enables it by listing its directory under `components`.

A Component is more than a folder of YAML. It participates in the same Kustomize build as its consumer:

1. Kustomize accumulates the consumer's resources (including its base).
2. It accumulates each selected Component in listed order.
3. A Component can add resources of its own and patch resources already accumulated by the consumer or an earlier Component.
4. Kustomize applies the combined transformations and emits one manifest stream.

That means `pod-security` can patch the base Deployment without copying that Deployment into the Component. `network-policy` contributes a new resource. Both are opt-in, so `dev` remains unchanged and `prod` opts in to both. A regular overlay is usually the right place for environment identity and replica counts; a Component is a good fit for a reusable, optional capability.

Inspect the Component definitions:

- `../examples/components/pod-security/kustomization.yaml` and `deployment-patch.yaml`
- `../examples/components/network-policy/kustomization.yaml` and `network-policy.yaml`

Now render production:

```sh
kubectl kustomize doc/examples/apps/catalog/overlays/prod
```

Confirm the output has three replicas, the pod security settings, and a NetworkPolicy. `../examples/apps/catalog/overlays/prod/kustomization.yaml` selects these Components; `dev` does not. The component paths are relative to the kustomization file that lists them.

### Component habits that scale

- Keep a Component focused on one capability, and give it a stable directory and clear name.
- Put optional resources and the patches they need together. This keeps enablement to one component reference.
- Patch by resource identity and container name; avoid brittle array indexes when a strategic-merge patch can express the intent.
- Build Components against stable labels and names provided by the base. Keep environment-specific values in overlays.
- Treat component order as meaningful when Components modify the same field or resource. Prefer independent Components so order does not become a hidden dependency.
- Make opt-in explicit. Missing component directories fail the build by default; do not enable `ignoreMissingComponents` to hide a typo.
- Render each overlay after changing a Component. A Component can affect every consumer that opts into it.

