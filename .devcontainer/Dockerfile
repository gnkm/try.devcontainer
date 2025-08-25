# Simplified, reliable Dockerfile for development
FROM node:20-bullseye-slim

# Set timezone
ARG TZ=UTC
ENV TZ="$TZ"

# Define versions
ARG CLAUDE_CODE_VERSION=latest
ARG GIT_DELTA_VERSION=0.18.2

# Install system packages in a single layer
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Essential tools
    git \
    curl \
    wget \
    ca-certificates \
    gnupg2 \
    # Development tools
    sudo \
    vim \
    nano \
    less \
    procps \
    unzip \
    jq \
    # Python and development
    python3 \
    python3-pip \
    python3-venv \
    # Shell improvements
    zsh \
    fzf \
    # GitHub CLI
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt-get update \
    && apt-get install -y gh \
    # Cleanup
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install git-delta
RUN ARCH=$(dpkg --print-architecture) && \
    wget -q "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
    rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Create directories and set permissions
RUN mkdir -p /workspace /usr/local/share/npm-global /home/node/.local/bin && \
    chown -R node:node /workspace /usr/local/share/npm-global /home/node

# Switch to node user
USER node
WORKDIR /workspace

# Configure npm global directory
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=/usr/local/share/npm-global/bin:/home/node/.local/bin:$PATH

# Install Claude Code
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}

# Install Python development tools
RUN pip3 install --user --no-cache-dir \
    ruff \
    mypy \
    pytest \
    uv

# Set up basic git configuration
RUN git config --global init.defaultBranch main && \
    git config --global user.email "devcontainer@example.com" && \
    git config --global user.name "Dev Container User"

# Configure zsh as default shell (simple setup without Oh My Zsh)
RUN echo 'export PATH="/usr/local/share/npm-global/bin:/home/node/.local/bin:$PATH"' > /home/node/.zshrc && \
    echo 'export SHELL=/usr/bin/zsh' >> /home/node/.zshrc && \
    echo 'autoload -U compinit && compinit' >> /home/node/.zshrc && \
    echo 'setopt AUTO_CD HIST_VERIFY HIST_IGNORE_DUPS' >> /home/node/.zshrc && \
    echo 'alias ll="ls -la"' >> /home/node/.zshrc && \
    echo 'alias la="ls -la"' >> /home/node/.zshrc

# Set environment variables
ENV SHELL=/usr/bin/zsh

# Verification and startup
CMD ["bash", "-c", "echo 'Development container ready!' && echo 'Installed tools:' && claude --version && python3 --version && ruff --version && delta --version && zsh"]