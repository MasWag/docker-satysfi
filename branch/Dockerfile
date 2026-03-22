FROM ghcr.io/maswag/satysfi-base-image:opam-2.5.0-ocaml-4.14.2 AS build-env

ARG BRANCH=master

RUN apt-get update && apt-get install -y bzip2 && rm -rf /var/lib/apt/lists/*
RUN opam update

# Install SATySFi and Satyrographos
RUN opam repository add satysfi-external https://github.com/gfngfn/satysfi-external-repo.git
RUN opam repository add satyrographos https://github.com/na4zagin3/satyrographos-repo.git
RUN opam update
RUN opam install satysfi satysfi-dist satyrographos
RUN eval $(opam env) && satysfi --version > /satysfi-revision && satyrographos --version > /satyrographos-revision

# Copy binaries
RUN eval $(opam env) && \
    satyrographos install --copy && \
    cp $(which satysfi) / && \
    cp $(which satyrographos) /

# --

FROM ubuntu:24.04

# Copy from build-env
COPY --from=build-env /root/.satysfi /usr/local/share/satysfi
COPY --from=build-env /satysfi /usr/local/bin/satysfi
COPY --from=build-env /satyrographos /usr/local/bin/satyrographos
COPY --from=build-env /satysfi-revision /satysfi-revision
COPY --from=build-env /satyrographos-revision /satyrographos-revision

RUN date --iso-8601=seconds > /build-date

# Check all shared libraries installed
RUN for bin in satysfi satyrographos; do \
    ldd $(which $bin) | grep "not found"; \
    if [ $? -eq 0 ]; then \
    exit 1; \
    fi; \
    done

# Setup build directory
RUN mkdir /satysfi
WORKDIR /satysfi

# OCI Annotations (https://github.com/opencontainers/image-spec/blob/main/annotations.md)
ARG BUILD_DATE
ARG VERSION
ARG REVISION
ARG BRANCH=master
ARG IMAGE_TAG
LABEL org.opencontainers.image.created=$BUILD_DATE \
    org.opencontainers.image.authors="amutake" \
    org.opencontainers.image.url="https://github.com/amutake/docker-satysfi" \
    org.opencontainers.image.documentation="https://github.com/amutake/docker-satysfi" \
    org.opencontainers.image.source="https://github.com/amutake/docker-satysfi" \
    org.opencontainers.image.version=$VERSION \
    org.opencontainers.image.revision=$REVISION \
    org.opencontainers.image.vendor="amutake" \
    org.opencontainers.image.licenses="Apache-2.0" \
    org.opencontainers.image.title="SATySFi ${BRANCH} branch and Satyrographos image without opam environment" \
    org.opencontainers.image.description="SATySFi ${BRANCH} branch and Satyrographos image without opam environment" \
    org.opencontainers.image.ref.name="amutake/satysfi:${IMAGE_TAG}"
