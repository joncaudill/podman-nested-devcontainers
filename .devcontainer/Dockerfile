FROM node:current-trixie

ENV DEBIAN_FRONTEND=noninteractive
ENV DISABLE_AUTOUPDATER=true

# Install system utilities, container tools, and target runtimes
# Don't remove podman, podman-compose, podman-docker, fuse-overlayfs, iptables, nftables, bash or it wont work
# The rest are there because they're a good "starter pack" to make developing easy
# (add any additional packages you need for your devcontiner in here)
RUN apt-get update && apt-get install -y --no-install-recommends \
    podman \
    podman-compose \
    podman-docker \
    fuse-overlayfs \
    iptables \
    nftables \
    bash \
    git \
    curl \
    zip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

## EVERYTHING ABOVE THIS LINE IS WHATEVER YOU NEED PER A NORMAL DOCKERFILE
## EVERYTHING BELOW THIS LINE EXCEPT THE WORKDIR SHOULDN'T BE EDITED

# Bake the nested storage configuration directly into the image
RUN mkdir -p /etc/containers /run/containers/storage /var/lib/containers/storage
RUN touch /etc/containers/nodocker

RUN cat <<EOF > /etc/containers/storage.conf
[storage]
driver = "overlay"
# If you define one, define both. 
# Graphroot is where the actual images/container files live.
runroot = "/run/containers/storage"
graphroot = "/var/lib/containers/storage"

[storage.options.overlay]
mount_program = "/usr/bin/fuse-overlayfs"
ignore_chown_errors = "true"
EOF

# Set the default working directory for the agent
WORKDIR /workspace