FROM mcr.microsoft.com/devcontainers/base:2.1.6-ubuntu24.04@sha256:3dcb059253b2ebb44de3936620e1cff3dadcd2c1c982d579081ca8128c1eb319

ARG DEBIAN_FRONTEND=noninteractive
ARG NODE_LTS_VERSION=24.14.1
ARG DOTNET_SDK_VERSION=10.0.201
ARG BUN_VERSION=1.3.11
ARG UV_VERSION=0.11.3
ARG PWSH_VERSION=7.6.0
ARG TERRAFORM_VERSION=1.14.8
ARG DEVCONTAINER_CLI_VERSION=0.85.0
ARG GEMINI_CLI_VERSION=0.36.0
ARG KIMI_CODE_VERSION=1.0.11
ARG CODEX_VERSION=0.118.0
ARG OPENCODE_VERSION=1.3.13
ARG GH_COPILOT_CLI_VERSION=1.0.16

# System dependencies (python3 needed for scripts/trace-extract.py)
RUN apt-get update && apt-get install -y --no-install-recommends \
      curl unzip git ca-certificates python3 \
    && rm -rf /var/lib/apt/lists/*

# Node.js (needed for MCP server packages)
RUN curl -fsSL "https://nodejs.org/dist/v${NODE_LTS_VERSION}/node-v${NODE_LTS_VERSION}-linux-x64.tar.xz" -o /tmp/node.tar.xz \
    && tar -xJf /tmp/node.tar.xz -C /usr/local --strip-components=1 \
    && rm /tmp/node.tar.xz

# .NET SDK 10.0.102 LTS
RUN curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin \
      --version "${DOTNET_SDK_VERSION}" \
      --install-dir /usr/share/dotnet \
    && ln -sf /usr/share/dotnet/dotnet /usr/local/bin/dotnet
ENV DOTNET_ROOT=/usr/share/dotnet
ENV PATH="${PATH}:${DOTNET_ROOT}:${DOTNET_ROOT}/tools"

# Avalonia templates
RUN dotnet new install Avalonia.Templates::11.3.13

# Bun
ENV BUN_INSTALL=/usr/local/bun
ENV PATH="${BUN_INSTALL}/bin:${PATH}"
RUN curl -fsSL https://bun.sh/install | bash -s -- bun-v${BUN_VERSION} \
    && ln -sf "${BUN_INSTALL}/bin/bun" /usr/local/bin/bun

# uv (Astral Python package manager)
RUN curl -LsSf https://astral.sh/uv/${UV_VERSION}/install.sh | sh \
    && cp /root/.local/bin/uv /usr/local/bin/uv \
    && cp /root/.local/bin/uvx /usr/local/bin/uvx \
    && chmod +x /usr/local/bin/uv /usr/local/bin/uvx

# GitHub CLI
RUN mkdir -p /etc/apt/keyrings \
    && chmod 755 /etc/apt/keyrings \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
       -o /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
       | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt-get update && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*

# PowerShell Core
RUN curl -fsSL "https://github.com/PowerShell/PowerShell/releases/download/v${PWSH_VERSION}/powershell-${PWSH_VERSION}-linux-x64.tar.gz" -o /tmp/pwsh.tar.gz \
    && mkdir -p /opt/microsoft/powershell/7 \
    && tar -xzf /tmp/pwsh.tar.gz -C /opt/microsoft/powershell/7 \
    && chmod +x /opt/microsoft/powershell/7/pwsh \
    && ln -sf /opt/microsoft/powershell/7/pwsh /usr/local/bin/pwsh \
    && rm /tmp/pwsh.tar.gz

# Terraform CLI
RUN curl -fsSL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" -o /tmp/terraform.zip \
    && unzip /tmp/terraform.zip -d /usr/local/bin \
    && chmod +x /usr/local/bin/terraform \
    && rm /tmp/terraform.zip

# Dev Container CLI
RUN bun install -g @devcontainers/cli@${DEVCONTAINER_CLI_VERSION}


# GitHub Copilot CLI (gh extension)
RUN gh extension install github/gh-copilot

# Aspire CLI
RUN curl -sSL https://aspire.dev/install.sh | bash \
    && cp /root/.aspire/bin/aspire /usr/local/bin/aspire \
    && chmod +x /usr/local/bin/aspire

# opencode CLI
RUN curl -fsSL https://opencode.ai/install | bash -s -- --version "${OPENCODE_VERSION}" --no-modify-path \
    && cp /root/.opencode/bin/opencode /usr/local/bin/opencode \
    && chmod +x /usr/local/bin/opencode

# Gemini CLI
RUN bun install -g @google/gemini-cli@${GEMINI_CLI_VERSION} --trust

# Kimi Code CLI
RUN bun install -g kimi-code@${KIMI_CODE_VERSION} --trust

# Codex CLI (OpenAI)
RUN bun install -g @openai/codex@${CODEX_VERSION} --trust

# GH Copilot CLI
RUN bun install -g @github/copilot@${GH_COPILOT_CLI_VERSION} --trust

# Validate all installed tools
RUN node --version \
    && dotnet --version \
    && bun --version \
    && uv --version \
    && gh --version \
    && pwsh --version \
    && terraform --version \
    && devcontainer --version \
    && gemini --version \
    && codex --version \
    && aspire --version \
    && opencode version \
    && python3 --version \
    && copilot --version
