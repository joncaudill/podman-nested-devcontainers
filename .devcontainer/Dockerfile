FROM alpine:latest

# 1. Install system utilities, container tools, and target runtimes
RUN apk add --no-cache \
    podman \
    fuse-overlayfs \
    iptables \
    nodejs \
    npm \
    bash \
    git

## EVERYTHING ABOVE THIS LINE IS WHATEVER YOU NEED PER A NORMAL DOCKERFILE
## EVERYTHING BELOW THIS LINE EXCEPT THE WORKDIR SHOULDN'T BE EDITED
# 2. Trick the agent's scripts by mapping the docker command to podman
RUN ln -s /usr/bin/podman /usr/bin/docker

# 3. Bake the nested storage configuration directly into the image
RUN mkdir -p /etc/containers
RUN echo -e '[storage]\ndriver = "overlay"\n[storage.options.overlay]\nmount_program = "/usr/bin/fuse-overlayfs"\nignore_chown_errors = "true"' > /etc/containers/storage.conf

# Set the default working directory for the agent
WORKDIR /workspace