FROM ghcr.io/maswag/satysfi-base-image:opam-2.5.0-ocaml-4.14.2 AS build-env

# Install system dependencies
RUN apt-get update && apt-get install -y bzip2 && rm -rf /var/lib/apt/lists/*

# Install SATySFi and Saphe
RUN opam repository add satysfi-external https://github.com/gfngfn/satysfi-external-repo.git
RUN opam update
RUN opam install saphe satysfi

# Generate metadata
RUN eval $(opam env) && satysfi --version > /satysfi-revision
RUN date --iso-8601=seconds > /build-date

# Copy binaries
# NOTE:
#   This is a dynamic link binary, but there is no problem because the same shared
#   libraries are included even at the runtime stage. check `ldd $(which satysfi)`
RUN eval $(opam env) && \
    cp $(which saphe) / && \
    cp $(which satysfi) /

# --

FROM ubuntu:24.04

# Copy from build-env
COPY --from=build-env /saphe /usr/local/bin/saphe
COPY --from=build-env /satysfi /usr/local/bin/satysfi
COPY --from=build-env /satysfi-revision /satysfi-revision
COPY --from=build-env /build-date /build-date

# Setup runtime dependencies
RUN apt-get update && \
    apt-get install -y git wget unzip && \
    rm -rf /var/lib/apt/lists/*

# Check all shared libraries installed
RUN for bin in satysfi saphe; do \
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
    org.opencontainers.image.title="SATySFi and Saphe" \
    org.opencontainers.image.description="SATySFi and Saphe" \
    org.opencontainers.image.ref.name="amutake/satysfi:${IMAGE_TAG}"
