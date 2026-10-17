# application-manifests

Desired deploy state for Gurujix **application** workloads (image pins, env-specific values).

Platform cluster config (Argo Applications, observability, Kyverno, LBC, …) stays in
[`platform-gitops`](https://github.com/gurujix-tech/platform-gitops). App teams should not need write access there.

## Layout (now)

```text
service-orders/
  dev/values.yaml    # auto-promoted on every merge to service-orders main
  qa/values.yaml     # promoted only via Actions → Promote (workflow_dispatch)
```

## Releases + promote ladder

1. Feature branch → PR (all checks).
2. Merge to `service-orders` `main`.
3. CI **auto-bumps patch** (`v0.1.0` → `v0.1.1` → …), git tag, ECR publish, promote-PR → **`dev`**.
4. When ready for QA: GitHub Actions on `service-orders` → **Promote** → Run workflow  
   - `target_env`: `qa`  
   - `semver`: leave empty to copy whatever is in `dev`, or set e.g. `v0.1.3`  
   - Running the workflow **is** the gate; it opens + squash-merges the manifests PR.
5. Same semver later → `uat` / `prod` (add folders + workflow options; drop auto-merge for prod).

No rebuild on promote. No manual `git tag` for normal releases.

## Envs

| Env | Runtime | How pin updates |
| --- | --- | --- |
| **local** | kind | not in this repo |
| **dev** | EKS `default` ns | auto on main merge |
| **qa** | EKS `qa` ns (same cluster) | Promote workflow |
| **uat** / **prod** | later | Promote + stricter approval |

## Layout (later — multi-team + CODEOWNERS)

When **real** team boundaries exist (2+ teams), migrate — do not invent `teams/` early:

```text
teams/<team>/helm/<service>/<env>/values.yaml
CODEOWNERS
```

Until then: flat `<service>/<env>/` only.

## Bootstrap

1. Repo `gurujix-tech/application-manifests` with this tree.
2. Argo CD can fetch the repo (public or registered credentials).
3. `service-orders` secret `APPLICATION_MANIFESTS_TOKEN` — Contents + Pull requests R/W on this repo.
4. Push `platform-gitops` so `apps-eks/service-orders-qa.yaml` is synced.
