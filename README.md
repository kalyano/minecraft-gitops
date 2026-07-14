# Minecraft on Akuity (Kargo + Argo CD)

My run through the Akuity Kargo quickstart! Instead of the sample guestbook app,
I deploy a Minecraft server and promote new versions from staging to prod. Argo CD
does the deploying, Kargo handles the promotions. Running on a local kind cluster
with a self-hosted agent.

## Layout

```
app/       Minecraft workload - Kustomize base + staging/prod overlays
kargo/     Kargo project, warehouse (watches the image), and the staging -> prod stages
argocd/    one Argo CD Application per environment
```

## How it works

- The warehouse watches `itzg/minecraft-server` on Docker Hub for new SemVer tags.
- Promoting a stage makes Kargo bump the image tag in that environment's overlay,
  commit it, and tell Argo CD to sync. Git stays the source of truth.
- staging pulls straight from the warehouse. prod only accepts what staging has
  already run, so nothing skips the line.

## Notes and decisions

- Two stages (staging, prod) to keep it simple to demo. Same pattern extends to more
  environments or an ApplicationSet.
- Went with Minecraft because it's stateful (single replica, Recreate strategy, slow
  startup so it needs a real startup probe) and the image has clean semver tags. Makes
  a better test than a stateless web app, and you can see the MOTD change per env.
- World storage is `emptyDir`, so it's disposable. Would be a PVC for anything real.
- prod sync is manual, staging auto-syncs.
- Manifests use the current Kargo 1.x syntax (`promotionTemplate` steps), not the
  older `promotionMechanisms` the published tutorial still uses.