FROM ubuntu:20.04 as env

ARG GITHUB_RUNNER_VERSION=2.299.1
ARG DEBIAN_FRONTEND=noninteractive

WORKDIR /root
RUN apt-get update && apt-get install wget -y
RUN wget https://github.com/actions/runner/releases/download/v${GITHUB_RUNNER_VERSION}/actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz \
    && tar xzf ./actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz && rm -f actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz

FROM ubuntu:20.04 as runner

ARG DEBIAN_FRONTEND=noninteractive
ENV KMS_SERVER_ADDR ""
ENV RUNNER_REGISTER_TO ""
ENV RUNNER_WORKDIR "_work"
ENV RUNNER_LABELS ""
ENV ADDITIONAL_FLAGS ""

# Basic setup of users and packages. This section is shamelessly taken from:
# https://github.com/myoung34/docker-github-actions-runner/blob/adff5c97f33632d945b424f053563e168e9fdd9d/Dockerfile.base

ENV DOCKER_COMPOSE_VERSION="1.27.4"
ENV GIT_LFS_VERSION="3.2.0"

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
# hadolint ignore=SC2086,DL3015,DL3008,DL3013


RUN echo en_US.UTF-8 UTF-8 >> /etc/locale.gen \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    gnupg \
    lsb-release \
    curl \
    tar \
    unzip \
    apt-transport-https \
    ca-certificates \
    sudo \
    gpg-agent \
    software-properties-common \
    build-essential \
    zlib1g-dev \
    zstd \
    gettext \
    libcurl4-openssl-dev \
    inetutils-ping \
    jq \
    wget \
    dirmngr \
    openssh-client \
    locales \
    python3-pip \
    python3-setuptools \
    python3 \
    dumb-init \
    rsync \
    gosu \
    apt-utils

RUN apt-get install -y \
    git-core \
    git \
    git-lfs

## Cleanup etc is required for nodejs, npm and yarn to get installe
RUN apt-get remove --purge nodejs npm \
    && apt-get clean \
    && apt-get autoclean \
    && apt-get install -f \
    && apt-get autoremove \
    && curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - \
    && apt-get install -y nodejs yarn

RUN DPKG_ARCH="$(dpkg --print-architecture)" \
  && LSB_RELEASE_CODENAME="$(lsb_release --codename | cut -f2)" \
  && sed -e 's/Defaults.*env_reset/Defaults env_keep = "HTTP_PROXY HTTPS_PROXY NO_PROXY FTP_PROXY http_proxy https_proxy no_proxy ftp_proxy"/' -i /etc/sudoers \
  && apt-get update \
  && ( [[ $(apt-cache search -n liblttng-ust0 | awk '{print $1}') == "liblttng-ust0" ]] && apt-get install -y --no-install-recommends liblttng-ust0 || : ) \
  && ( [[ $(apt-cache search -n liblttng-ust1 | awk '{print $1}') == "liblttng-ust1" ]] && apt-get install -y --no-install-recommends liblttng-ust1 || : ) \
  && ( ( curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" && unzip awscliv2.zip -d /tmp/ && /tmp/aws/install && rm awscliv2.zip) || pip3 install --no-cache-dir awscli ) \
  && distro=$(lsb_release -is | awk '{print tolower($0)}') \
  && mkdir -p /etc/apt/keyrings \
  && ( curl -fsSL https://download.docker.com/linux/${distro}/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg ) \
  && version=$(lsb_release -cs | sed 's/bookworm\|n\/a/bullseye/g') \
  && ( echo "deb [arch=${DPKG_ARCH} signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/${distro} ${version} stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null ) \
  && apt-get update \
  && apt-get install -y docker-ce docker-ce-cli containerd.io --no-install-recommends --allow-unauthenticated \
  && ( [[ $(lscpu -J | jq -r '.lscpu[] | select(.field == "Vendor ID:") | .data') == "ARM" ]] && echo "Not installing docker-compose. See https://github.com/docker/compose/issues/6831" || ( curl -sL "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose ) ) \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "focal" ]] && ( echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list && curl -L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key" | apt-key add - && apt-get update) || : ) \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "focal" || "${LSB_RELEASE_CODENAME}" == "jammy" || "${LSB_RELEASE_CODENAME}" == "sid" || "${LSB_RELEASE_CODENAME}" == "bullseye" ]] && apt-get install -y --no-install-recommends podman buildah skopeo || : ) \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "jammy" ]] && echo "Ubuntu Jammy is marked as beta. Please see https://github.com/actions/virtual-environments/issues/5490" || : ) \
  && GH_CLI_VERSION=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq -r '.tag_name' | sed 's/^v//g') \
  && GH_CLI_DOWNLOAD_URL=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq ".assets[] | select(.name == \"gh_${GH_CLI_VERSION}_linux_${DPKG_ARCH}.deb\")" | jq -r '.browser_download_url') \
  && curl -sSLo /tmp/ghcli.deb ${GH_CLI_DOWNLOAD_URL} && apt-get -y install /tmp/ghcli.deb && rm /tmp/ghcli.deb \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /tmp/* \
  && groupadd -g 121 runner \
  && useradd -mr -d /home/runner -u 1001 -g 121 runner \
  && usermod -aG sudo runner \
  && usermod -aG docker runner \
  && echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

COPY --from=env /root/ /home/runner/
RUN  /home/runner/bin/installdependencies.sh

WORKDIR /home/runner/

COPY entrypoint.sh runsvc.sh ./
RUN chmod u+x ./entrypoint.sh ./runsvc.sh

RUN chown -R runner:runner /home/runner
USER runner

RUN install -m 600 -D /dev/null ~/.ssh/known_hosts
RUN ssh-keyscan -H github.com >> ~/.ssh/known_hosts

ENTRYPOINT ["./entrypoint.sh"]
