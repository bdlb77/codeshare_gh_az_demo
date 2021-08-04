#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

# [Choice] Node.js version: 16, 14, 12
ARG VARIANT=16
FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:${VARIANT}
# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# The node image includes a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=node
ARG USER_UID=1000
ARG USER_GID=$USER_UID


# copy the stup scripts to the container
COPY script-library/*.sh /scripts/

# install selected components
# many scripts take optional parameters
# see the script header for details of each
RUN export DEBIAN_FRONTEND=noninteractive apt-get update \
  # order matters for the next 4 scripts
  # uncomment this line if you're using a base image that is not from vscode/devcontainers
  #&& /bin/bash /scripts/common-debian.sh \
  && /bin/bash /scripts/docker-in-docker-debian.sh \
  && /bin/bash /scripts/kubectl-helm-debian.sh \
  && /bin/bash /scripts/kind-k3d-debian.sh \
  # set welcome message
  && echo "👋 Welcome to Codespaces! You are on a custom image defined in your devcontainer.json file.\n" > /usr/local/etc/vscode-dev-containers/first-run-notice.txt \
  && echo "🔍 To explore VS Code to its fullest, search using the Command Palette (Cmd/Ctrl + Shift + P)\n" >> /usr/local/etc/vscode-dev-containers/first-run-notice.txt \
  && echo "run 'make all' to build a kind cluster in Codespaces\n\n" >> /usr/local/etc/vscode-dev-containers/first-run-notice.txt

# Configure apt and install packages
RUN apt-get update \
  && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
  #
  # Verify git and needed tools are installed
  && apt-get -y install git iproute2 procps \
  #
  # Remove outdated yarn from /opt and install via package
  # so it can be easily updated via apt-get upgrade yarn
  && rm -rf /opt/yarn-* \
  && rm -f /usr/local/bin/yarn \
  && rm -f /usr/local/bin/yarnpkg \
  && apt-get install -y curl apt-transport-https lsb-release \
  && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
  && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
  && apt-get update \
  && apt-get -y install --no-install-recommends yarn \
  #
  # Install eslint globally
  && npm install -g eslint \
  #
  # [Optional] Update a non-root user to UID/GID if needed.
  && if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
  groupmod --gid $USER_GID $USERNAME \
  && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
  && chown -R $USER_UID:$USER_GID /home/$USERNAME; \
  fi \
  # [Optional] Add add sudo support for non-root user
  && apt-get install -y sudo \
  && echo node ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
  && chmod 0440 /etc/sudoers.d/$USERNAME \
  #
  # Clean up
  && apt-get autoremove -y \
  && apt-get clean -y \
  && rm -rf /var/lib/apt/lists/*



# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# docker socket required for Docker in Docker
VOLUME [ "/var/lib/docker" ]

# Setting the ENTRYPOINT to docker-init.sh will start up the Docker Engine
# inside the container "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]

