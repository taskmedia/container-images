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

| Image | ghcr.io | Docker Hub |
|-------|---------|------------|
| [kubectl-yq](images/kubectl-yq) | `ghcr.io/taskmedia/kubectl-yq` | `fty4/kubectl-yq` |
| [kubectl-gpg-ncftp](images/kubectl-gpg-ncftp) | `ghcr.io/taskmedia/kubectl-gpg-ncftp` | `fty4/kubectl-gpg-ncftp` |
| [curl-jq](images/curl-jq) | `ghcr.io/taskmedia/curl-jq` | `fty4/curl-jq` |
| [scratch-success](images/scratch-success) | `ghcr.io/taskmedia/scratch-success` | `fty4/scratch-success` |

Each image was originally maintained in its own repository; those repositories
have been merged in with full git history preserved and are now archived,
each with a README pointing back here.
