# platform-gitops

What runs in each cluster of the card onboarding estate, declared in Git and applied by **Flux** (ADR-0021 in card-onboarding-platform). A change to a cluster is a pull request here; Flux pulls this public repository every two minutes and makes the cluster match.

## Layout

| Path | What |
| --- | --- |
| `clusters/cards-pilot/flux-system/` | Flux v2.10.0, vendored from its release manifest. Only source, kustomize, helm and notification controllers run; the image-automation controllers are removed to save memory. Also the sync of this repo |
| `clusters/cards-pilot/infrastructure.yaml` | Kustomization `infrastructure` → `./infrastructure` |
| `clusters/cards-pilot/apps-dev.yaml` | Kustomization `apps-dev` → `./apps/dev`, after `infrastructure` |
| `infrastructure/` | Sources (the team repos, the chart registries), cert-manager, the Envoy Gateway controller, the API gateway (routes and policies) |
| `apps/dev/` | otp-service (comms-dev) and channel-web (channels-dev), images pinned by commit |

The releases keep the names `make` installed them with (`eg`, `api-gateway`, `cert-manager`, `otp-service`, `channel-web`), so Flux adopted them in place.

## Promote a change

1. A team merges to `main`, and its CI publishes `ghcr.io/cards-pilot-estate/<image>:<sha>`; the images are public.
2. Open a PR here that changes `image.tag` in `apps/dev/<service>.yaml` to that commit.
3. After merge, Flux rolls it out within about 2 minutes: `kubectl -n flux-system get kustomizations,helmreleases -A`.

Charts follow each team repo's `main` in dev. uat and prod will pin chart revisions and image tags (`apps/uat`, `apps/prod`).

## Secrets

No secret is stored in Git. Each team creates its own once per environment (`make secrets ENV=<env>` in sim-comms and channel-web; `make iam`, `make platform` and `make secrets` in the platform repo), and the charts refer to them by name. Encrypting secrets into Git with SOPS and age is the next step, when a secret has to travel with the configuration (the design's R8).

## Not yet here

The platform itself (onboarding-service, bank-simulators, ops-console, Postgres, RabbitMQ, Redis, Keycloak, observability) is still deployed by the platform repository's CD workflow on the self-hosted runner. That repository is private, and its images live in a private registry. It moves here when it is public, or with a read-only pull credential (WG.11b).

## Bootstrap a cluster

```bash
kubectl apply -k clusters/cards-pilot/flux-system      # installs Flux and points it at this repo
kubectl -n flux-system wait --for=condition=Ready kustomization/flux-system --timeout=5m
```
