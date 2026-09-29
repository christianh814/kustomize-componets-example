# Walk: add reusable Components

A **Component** is an opt-in Kustomize package for a capability that can extend or transform resources assembled by a base or overlay. Its `kustomization.yaml` uses `kind: Component`. The consumer enables it by listing its directory under `components`.

A Component is more than a folder of YAML. It participates in the same Kustomize build as its consumer:

1. Kustomize accumulates the consumer's resources (including its base).
2. It accumulates each selected Component in listed order.
3. A Component can add resources of its own and patch resources already accumulated by the consumer or an earlier Component.
4. Kustomize applies the combined transformations and emits one manifest stream.

That means `pod-security` can patch the base Deployment without copying that Deployment into the Component. `network-policy` contributes a new resource. Both are opt-in, so `dev` remains unchanged and `prod` opts in to both. A regular overlay is usually the right place for environment identity and replica counts; a Component is a good fit for a reusable, optional capability.

## Overlay or Component?

Both can use Kustomize patches and resources, but they play different roles:

| | Overlay | Component |
|---|---|---|
| Purpose | Defines a complete variant to build and deploy | Packages an optional capability for an overlay to opt into |
| Typical choice | `dev`, `staging`, `prod`, a region, or a customer deployment | Pod security, ingress, monitoring, policy, or an optional integration |
| How it is used | Argo CD points `spec.source.path` at the overlay | The overlay lists it under `components`, or Argo CD adds it in `spec.source.kustomize.components` |
| Output | A complete set of manifests | No complete app by itself; it augments the consuming build |

An overlay answers **“which deployable variant is this?”** A Component answers **“which optional capability does this variant include?”** Use both together when, for example, production needs more replicas and a reusable security policy. Keep replica counts in the production overlay; put the opt-in security behavior in a Component.

Inspect the Component definitions:

- [`pod-security`](../examples/components/pod-security/kustomization.yaml) and its `deployment-patch.yaml`
- [`network-policy`](../examples/components/network-policy/kustomization.yaml) and its `network-policy.yaml`

Now render production:

```sh
kubectl kustomize doc/examples/apps/catalog/overlays/prod
```

Confirm the output has three replicas, the pod security settings, and a NetworkPolicy. [`prod/kustomization.yaml`](../examples/apps/catalog/overlays/prod/kustomization.yaml) selects these Components; `dev` does not. The component paths are relative to the kustomization file that lists them.

### Component habits that scale

- Keep a Component focused on one capability, and give it a stable directory and clear name.
- Put optional resources and the patches they need together. This keeps enablement to one component reference.
- Patch by resource identity and container name; avoid brittle array indexes when a strategic-merge patch can express the intent.
- Build Components against stable labels and names provided by the base. Keep environment-specific values in overlays.
- Treat component order as meaningful when Components modify the same field or resource. Prefer independent Components so order does not become a hidden dependency.
- Make opt-in explicit. Missing component directories fail the build by default; do not enable `ignoreMissingComponents` to hide a typo.
- Render each overlay after changing a Component. A Component can affect every consumer that opts into it.

### When to choose each

Choose an **overlay** when a difference creates a deployable target of its own: development versus production replica counts, a region-specific endpoint, or a customer-specific configuration. Each Argo CD Application can point at the appropriate overlay.

Choose a **Component** when the difference is an optional, reusable feature that can be switched on for one or more overlays: a security hardening patch, an ingress plus its supporting resources, or telemetry configuration. Components help avoid making a separate overlay for every combination of features.

If every deployment always needs a setting, put it in the base. If a value is specific to one environment, keep it in that overlay. If a reusable feature should be enabled selectively, make it a Component.
