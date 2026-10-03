# container-images

Shared OCI container images maintained by task.media. Each image lives under
its own directory in `images/` with a `Dockerfile` and image-specific
`README.md`, and is built/published by a single shared dynamic CI workflow
(`.github/workflows/docker.yml`, driven by `.github/images.yml`).

Images are built on a biweekly schedule, on `workflow_dispatch`, and on every
push/PR that touches an image's own directory (or a shared/common file, which
rebuilds everything). Each image publishes to the same coordinates it always
has:

```bash
# Pull from ghcr.io (GitHub Container Registry)
docker pull ghcr.io/taskmedia/<name>:main

# Pull from Docker Hub
docker pull fty4/<name>:main
```

## Images

| Image | ghcr.io | Docker Hub | Status |
|-------|---------|------------|--------|
| kubectl-yq | `ghcr.io/taskmedia/kubectl-yq` | `fty4/kubectl-yq` | pending merge |
| kubectl-gpg-ncftp | `ghcr.io/taskmedia/kubectl-gpg-ncftp` | `fty4/kubectl-gpg-ncftp` | pending merge |
| curl-jq | `ghcr.io/taskmedia/curl-jq` | `fty4/curl-jq` | pending merge |
| scratch-success | `ghcr.io/taskmedia/scratch-success` | `fty4/scratch-success` | pending merge |

Each image was originally maintained in its own repository; those repositories
are being merged in with full git history preserved and will be archived with
a pointer back here once the move is complete.
