FROM alpine:3.20

# Image annotations
# see: https://github.com/opencontainers/image-spec/blob/main/annotations.md#pre-defined-annotation-keys
LABEL org.opencontainers.image.title=kubectl-yq
LABEL org.opencontainers.image.description="image with kubectl and yq installed"
LABEL org.opencontainers.image.url=https://github.com/taskmedia/kubectl-yq/pkgs/container/kubectl-yq
LABEL org.opencontainers.image.source=https://github.com/taskmedia/kubectl-yq/blob/main/Dockerfile
LABEL org.opencontainers.image.vendor=task.media
LABEL org.opencontainers.image.licenses=MIT

# Install deps
RUN apk add --no-cache curl ca-certificates bash jq tar kubectl

RUN LATEST_VERSION=$(curl -s https://api.github.com/repos/mikefarah/yq/releases/latest | jq -r '.tag_name') \
  && ARCH=$(apk --print-arch) \
  && case "$ARCH" in \
       x86_64)   YARCH=amd64 ;; \
       aarch64)  YARCH=arm64 ;; \
       armhf)    YARCH=arm ;; \
       armv7)    YARCH=arm ;; \
       *) echo "unsupported arch: $ARCH" && exit 1 ;; \
     esac \
  && curl -L "https://github.com/mikefarah/yq/releases/download/${LATEST_VERSION}/yq_linux_${YARCH}.tar.gz" \
  | tar xz \
  && mv yq_linux_* /usr/bin/yq

# nobody
USER 65534

# print versions
RUN kubectl version --client && yq --version
