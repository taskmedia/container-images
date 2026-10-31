FROM kanboard/kanboard:v1.2.56

# Image annotations
# see: https://github.com/opencontainers/image-spec/blob/main/annotations.md#pre-defined-annotation-keys
LABEL org.opencontainers.image.title=kanboard
LABEL org.opencontainers.image.description="kanboard with Alpine php84 packages upgraded to the latest patch release"
LABEL org.opencontainers.image.url=https://github.com/taskmedia/container-images/pkgs/container/kanboard
LABEL org.opencontainers.image.source=https://github.com/taskmedia/container-images/blob/main/images/kanboard/Dockerfile
LABEL org.opencontainers.image.vendor=task.media
LABEL org.opencontainers.image.licenses=MIT

# upstream installs php84 unpinned at its own build time and rebuilds rarely;
# upgrading here picks up fixes such as PHP2Shell (CVE-2026-109618/-109619, fixed in 8.4.27)
RUN apk upgrade --no-cache $(apk info | grep -E '^php84(-|$)') \
    && php -v
