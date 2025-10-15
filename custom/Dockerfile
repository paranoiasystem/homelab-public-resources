ARG BASE_IMAGE=ubuntu:20.04
ARG KAIROS_INIT=v0.6.0-RC1

FROM quay.io/kairos/kairos-init:${KAIROS_INIT} AS kairos-init

FROM ${BASE_IMAGE} AS base-kairos
ARG MODEL=generic
ARG TRUSTED_BOOT=false
ARG KUBERNETES_DISTRO
ARG KUBERNETES_VERSION
ARG VERSION

LABEL org.opencontainers.image.source="https://github.com/paranoiasystem/homelab-public-resources"

RUN apt-get update && apt-get upgrade -y

# Install cifs-utils
RUN apt-get install -y cifs-utils

# Install nfs-common
RUN apt-get install -y nfs-common

# Install iSCSI tools
RUN apt-get install -y open-iscsi lsscsi sg3-utils multipath-tools scsitools
COPY custom/multipath.conf /etc/multipath.conf

RUN ln -s /lib/systemd/system/multipath-tools.service /etc/systemd/system/multi-user.target.wants/multipath-tools.service || true

RUN ln -s /lib/systemd/system/open-iscsi.service /etc/systemd/system/multi-user.target.wants/open-iscsi.service || true

RUN --mount=type=bind,from=kairos-init,src=/kairos-init,dst=/kairos-init \
    if [ -n "${KUBERNETES_DISTRO}" ]; then \
        K8S_FLAG="-p ${KUBERNETES_DISTRO}"; \
        if [ "${KUBERNETES_DISTRO}" = "k0s" ] && [ -n "${KUBERNETES_VERSION}" ]; then \
            K8S_VERSION_FLAG="--provider-k0s-version \"${KUBERNETES_VERSION}\""; \
        elif [ "${KUBERNETES_DISTRO}" = "k3s" ] && [ -n "${KUBERNETES_VERSION}" ]; then \
            K8S_VERSION_FLAG="--provider-k3s-version \"${KUBERNETES_VERSION}\""; \
        else \
            K8S_VERSION_FLAG=""; \
        fi; \
    else \
        K8S_FLAG=""; \
        K8S_VERSION_FLAG=""; \
    fi; \
    eval /kairos-init -l debug -s install -m \"${MODEL}\" -t \"${TRUSTED_BOOT}\" ${K8S_FLAG} ${K8S_VERSION_FLAG} --version \"${VERSION}\" && \
    eval /kairos-init -l debug -s init -m \"${MODEL}\" -t \"${TRUSTED_BOOT}\" ${K8S_FLAG} ${K8S_VERSION_FLAG} --version \"${VERSION}\"