# application-manifests

Desired deploy state for Gurujix **application** workloads (image pins, env-specific values).

Platform cluster config (Argo Applications, observability, Kyverno, LBC, …) stays in
[`platform-gitops`](https://github.com/gurujix-tech/platform-gitops). App teams should not need write access there.

## Layout (now)

```text
service-orders/
  dev/values.yaml    # image.tag = semver (vMAJOR.MINOR.PATCH), set by release tag CI
```

## Releases (semver)

1. Feature branch → PR (all checks).
2. Merge to `service-orders` `main`.
3. CI **auto-bumps patch** (`v0.1.0` → `v0.1.1` → …), creates the git tag, publishes ECR, promote-PR to `dev/values.yaml`.
4. You do **not** tag by hand. Later envs: promote the **same** semver (dispatch + approve) — no rebuild.

To bump **minor/major** later: add a workflow input or label; until then every main merge is patch +1.


## Layout (later — multi-team + CODEOWNERS)

When **real** team boundaries exist (2+ teams), migrate — do not invent `teams/` early:

```text
teams/<team>/helm/<service>/<env>/values.yaml
CODEOWNERS
```

Example `CODEOWNERS`:

```text
*                       @gurujix-tech/platform
teams/checkout/         @gurujix-tech/checkout
```

Then: protect `main` (PR + required CODEOWNERS review); point Argo `path:` and CI promote at the new paths; keep app-team tokens scoped to this repo only (never `platform-gitops` write).

Until then: flat `<service>/<env>/` only.

## Envs

| Env | Runtime |
| --- | --- |
| **local** | kind (not promoted here) |
| **dev** | EKS learning account |
| **staging** / **prod** | later |

## Bootstrap

1. Create GitHub repo `gurujix-tech/application-manifests` and push this tree.
2. Register the repo in Argo CD on EKS (same credentials pattern as other private repos).
3. In `service-orders`, set secret `APPLICATION_MANIFESTS_TOKEN` (fine-grained PAT on this repo with
   **Contents: Read and write** and **Pull requests: Read and write**).
4. Promotes from `service-orders` CI open a PR (with image/SHA/run links) and squash-merge it — no direct push to `main`.
