ARG UBUNTU_VERSION="24.04"

FROM rust:bullseye AS builder

RUN rustup update && cargo install --locked zellij
RUN rustup update && cargo install --locked bob-nvim

FROM buildpack-deps:${UBUNTU_VERSION}-curl
ARG USERNAME="ubuntu"
ARG NVIM_CONFIG="none"
ARG NVIM_VERSION="stable"

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends           \
    # dependencies
    gcc             \
    build-essential \
    make            \
    unzip           \
    git             \
    # Tools
    ripgrep         \
    fd-find         \
    fzf             \
    xclip           \
    ;               \
    # Clear cache 
    apt-get dist-clean 

RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://repo.charm.sh/apt/gpg.key | gpg --dearmor -o /etc/apt/keyrings/charm.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/charm.gpg] https://repo.charm.sh/apt/ * *" | tee /etc/apt/sources.list.d/charm.list && \
    apt update && apt install gum glow

# Create the user if they don't exist
RUN id -u ${USERNAME} 2>/dev/null || useradd -ms /bin/bash ${USERNAME}
# Add the user to the sudo group if not already in it
RUN id -nG ${USERNAME} | grep -qw sudo || sudo usermod -aG sudo ${USERNAME}

USER ${USERNAME}

COPY --from=builder /usr/local/cargo/bin/bob /usr/local/bin/
ENV PATH="/home/${USERNAME}/.local/share/bob/nvim-bin/:${PATH}"

# Fix issue that can crashes zellij. https://github.com/zellij-org/zellij/issues/3617 
ENV ZELLIJ_SOCKET_DIR="/tmp/zellij"
COPY --from=builder /usr/local/cargo/bin/zellij /usr/local/bin/

# Install nvim config
RUN if [ "${NVIM_CONFIG}" != "none" ]; then   \
        git clone ${NVIM_CONFIG} "${XDG_CONFIG_HOME:-$HOME/.config}"/nvim; \
    fi

# Set the version of nvim
RUN if [ "${NVIM_VERSION}" != "none" ]; then   \
        bob use ${NVIM_VERSION}; \
    fi