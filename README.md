# storycover-config

Desired state for the StoryCover app, managed with Kustomize. Argo CD reconciles this
repo onto the cluster; nothing here is applied by hand.

## Layout
```
storycover-config/
├── base/                     # environment-agnostic manifests
│   ├── serviceaccount.yaml   # storycover-app SA (Vault role is bound to it)
│   ├── deployment.yaml       # app pod: app node pool, restricted securityContext, probes
│   ├── service.yaml          # ClusterIP 80 -> 8080
│   ├── hpa.yaml              # CPU-based autoscaling
│   ├── networkpolicy.yaml    # default-deny, egress to DNS/Vault/HTTPS only
│   └── kustomization.yaml
└── overlays/
    ├── dev/                  # replicas 1, dev bucket, dev image (CI bumps the tag)
    └── prod/                 # replicas 2, prod bucket
```

## Branches are environments
`main` is dev, `prod` is production. App CI bumps the image tag in `overlays/dev` on
`main`; promotion to production is a `main -> prod` merge. Argo CD tracks `main` for the
dev app and `prod` for the prod app.

## The Gemini secret
The container reads `GEMINI_API_KEY` from the `gemini-api-key` Secret (KV field `api_key`).
That Secret is produced by the Vault Secrets Operator from a `VaultStaticSecret` owned by
the infra repo; the `storycover` namespace and the VaultStaticSecret are not defined here.
No secret value lives in this repo, only the reference.

## Notes
- `GCS_BUCKET` and image tags are the only per-env differences; they live in the overlays.
- Argo CD Application/Project wiring for this repo is handled separately and is still being
  decided, so it is not committed here yet.
- Prod image registry and project scoping are not settled; the prod overlay points at the
  dev registry as a placeholder until promotion is designed.
