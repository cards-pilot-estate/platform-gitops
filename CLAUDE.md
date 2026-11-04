# Working on this repository (assisted sessions)

Read first: `README.md`, then ADR-0021 and `docs/channels/delivery-and-environments.md` in card-onboarding-platform.

## Rules
- Every change is a pull request with green CI (`kustomize build` and schema validation); squash merge. Flux applies `main`.
- Keep the release names and namespaces the services already use (Flux adopts releases by name).
- Pin images by commit sha in `apps/<env>`; never `latest` or `main`.
- No secrets in Git (not even encoded). SOPS with age, when introduced, is the only way.
- No real bank or vendor names.
