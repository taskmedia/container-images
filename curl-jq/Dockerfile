FROM curlimages/curl

# Image annotations
# see: https://github.com/opencontainers/image-spec/blob/main/annotations.md#pre-defined-annotation-keys
LABEL org.opencontainers.image.title=curl-jq
LABEL org.opencontainers.image.description="image with curl and jq installed"
LABEL org.opencontainers.image.url=https://github.com/taskmedia/curl-jq/pkgs/container/curl-jq
LABEL org.opencontainers.image.source=https://github.com/taskmedia/curl-jq/blob/main/Dockerfile
LABEL org.opencontainers.image.vendor=task.media
LABEL org.opencontainers.image.licenses=MIT

USER root

RUN apk add jq

# curl_user
USER 100

# set workdirectory from / to /home/curl_user
WORKDIR /home/curl_user

# print versions
RUN curl --version && jq --version
