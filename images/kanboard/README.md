# Docker image kanboard

[Kanboard](https://github.com/kanboard/kanboard) upstream image with the
Alpine `php84*` packages upgraded to the newest available patch release.

Upstream installs PHP unpinned at its own build time and only rebuilds on new
Kanboard releases, so security fixes in PHP (e.g. PHP2Shell, fixed in 8.4.27)
can lag behind. This image rebuilds every two weeks and picks them up.

Drop this image once upstream ships a build with a patched PHP and switch back
to `kanboard/kanboard`.

Tags follow the upstream Kanboard tag the image is based on (first `FROM`
line of the Dockerfile):

| Tag | Meaning |
|-----|---------|
| `v1.2.55` | upstream version, moves with every rebuild (newest PHP patch) |
| `v1.2.55-20261021` | same, frozen at the build date; use this to pin |
| `main`, `latest` | most recent build |

```bash
# Pull from ghcr.io (GitHub Container Registry)
docker pull ghcr.io/taskmedia/kanboard:main

# Pull from Docker Hub
docker pull fty4/kanboard:main
```
