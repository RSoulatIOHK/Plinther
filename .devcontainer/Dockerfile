# Use Ubuntu base image
FROM --platform=linux/amd64 ubuntu:22.04

# Set environment to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    sudo \
    git \
    locales \
    xz-utils \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Set locale
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# Declare build-time variables
ARG USERNAME=vscode
ARG UID=1000
ARG GID=1000

# Create a non-root user with sudo privileges
RUN groupadd --gid ${GID} ${USERNAME} \
    && useradd --uid ${UID} --gid ${GID} -m ${USERNAME} \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

# Switch to the new user
USER ${USERNAME}
WORKDIR /home/${USERNAME}

# Install Nix in single-user mode
RUN curl -L https://nixos.org/nix/install | sh -s -- --no-daemon

# Set environment variables for Nix
ENV NIX_USER_PROFILE_DIR=/nix/var/nix/profiles/per-user/${USERNAME}
ENV PATH=/home/${USERNAME}/.nix-profile/bin:/home/${USERNAME}/.local/bin:${PATH}
ENV NIX_PATH=/nix/var/nix/profiles/per-user/${USERNAME}/channels

# Source Nix profile
SHELL ["/bin/bash", "-c"]
RUN . /home/${USERNAME}/.nix-profile/etc/profile.d/nix.sh

# Write nix.conf to the user's configuration directory
RUN mkdir -p /home/${USERNAME}/.config/nix \
    && echo 'build-users-group = nixbld' > /home/${USERNAME}/.config/nix/nix.conf \
    && echo 'substituters = https://cache.zw3rk.com https://cache.iog.io https://cache.nixos.org/' >> /home/${USERNAME}/.config/nix/nix.conf \
    && echo 'trusted-public-keys = loony-tools:pr9m4BkM/5/eSTZlkQyRt57Jz7OMBxNSUiMC4FkcNfk= \
              hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= \
              cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=' >> /home/${USERNAME}/.config/nix/nix.conf \
    && echo 'experimental-features = nix-command flakes' >> /home/${USERNAME}/.config/nix/nix.conf \
    && echo 'accept-flake-config = true' >> /home/${USERNAME}/.config/nix/nix.conf \
    && echo 'sandbox = false' >> /home/${USERNAME}/.config/nix/nix.conf

# Ensure the user owns the configuration directory
RUN chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.config

# Clone the minimal-ptt-examples repository
RUN git clone https://github.com/input-output-hk/minimal-ptt-examples.git /home/${USERNAME}/minimal-ptt-examples

# Set the working directory to the repository
WORKDIR /home/${USERNAME}/minimal-ptt-examples

# Pre-fetch dependencies using nix develop
RUN nix develop --command echo "Dependencies cached"

# Return to home directory
WORKDIR /home/${USERNAME}

# Set default shell
SHELL ["/bin/bash", "-c"]
CMD ["bash"]
