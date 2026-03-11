###############################################################################
# OpenClaw Gateway – Docker Image
#
# IMPORTANT: Runtime installs are lost on container restart. All binaries
# needed by skills MUST be installed here at build time. If you add a skill
# that needs a new binary, update this Dockerfile and rebuild.
###############################################################################

FROM node:22-bookworm-slim

# Install system dependencies as root
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    chromium \
    ca-certificates \
    gnupg \
    jq \
    ripgrep \
    tini \
  && rm -rf /var/lib/apt/lists/*

# Install GitHub CLI (required by github skill)
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
      | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
      > /etc/apt/sources.list.d/github-cli.list \
  && apt-get update && apt-get install -y --no-install-recommends gh \
  && rm -rf /var/lib/apt/lists/*

# Install gog CLI (required by gog skill — Google Workspace: Gmail, Calendar, Drive, etc.)
ARG GOG_VERSION=0.12.0
RUN ARCH=$(dpkg --print-architecture) \
  && curl -fsSL "https://github.com/steipete/gogcli/releases/download/v${GOG_VERSION}/gogcli_${GOG_VERSION}_linux_${ARCH}.tar.gz" \
     | tar -xz -C /usr/local/bin gog \
  && chmod +x /usr/local/bin/gog

# Pin OpenClaw version for reproducible builds — bump explicitly when upgrading
ARG OPENCLAW_VERSION=2026.3.8
RUN npm install -g openclaw@${OPENCLAW_VERSION}

# Install ClawHub CLI for skill management and summarize CLI (required by summarize skill)
RUN npm install -g clawhub @steipete/summarize

# Create OpenClaw directories owned by node
RUN mkdir -p /home/node/.openclaw/workspace /.clawhub \
  && chown -R node:node /home/node/.openclaw /.clawhub

# Copy skills manifest, workspace templates, and entrypoint
COPY config/skills-manifest.txt /opt/config/skills-manifest.txt
COPY workspace-templates/ /opt/workspace-templates/
COPY docker/entrypoint.sh /usr/local/bin/entrypoint.sh

# Pre-install extension dependencies at build time (optional, space-separated)
# Example: OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
ARG OPENCLAW_EXTENSIONS=""
RUN if [ -n "$OPENCLAW_EXTENSIONS" ]; then \
      OC_ROOT=$(npm root -g)/openclaw && \
      for ext in $OPENCLAW_EXTENSIONS; do \
        ext_dir="$OC_ROOT/extensions/$ext"; \
        if [ -f "$ext_dir/package.json" ]; then \
          echo "[build] Installing deps for extension: $ext"; \
          cd "$ext_dir" && npm install --omit=dev 2>/dev/null || true; \
        fi; \
      done; \
    fi

USER node

# Install Composio plugin (optional — activated by COMPOSIO_CONSUMER_KEY env var)
RUN openclaw plugins install @composio/openclaw-plugin

# Container health check (added in OpenClaw 2026.3.1)
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s --retries=3 \
  CMD curl -fsS http://127.0.0.1:18789/healthz || exit 1

ENTRYPOINT ["tini", "--", "entrypoint.sh"]
