# syntax=docker/dockerfile:1

ARG VARIANT="24"
FROM mcr.microsoft.com/devcontainers/javascript-node:${VARIANT}

# Install some generally useful tools
RUN apt-get update && apt-get -y install --no-install-recommends curl git sqlite3 bat

# Install SAP CAP SDK globally
USER node
RUN npm install -g @sap/cds-dk

# Create ~/.local/bin/ (as it's in the PATH) & symlink bat -> batcat
RUN mkdir -p "$HOME/.local/bin/" && ln -s "$(which batcat)" "$HOME/.local/bin/bat"

WORKDIR /home/node
