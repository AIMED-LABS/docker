FROM ubuntu:20.04 as env

ARG GITHUB_RUNNER_VERSION=2.299.1
ARG DEBIAN_FRONTEND=noninteractive

WORKDIR /root
RUN apt-get update && apt install wget -y
RUN wget https://github.com/actions/runner/releases/download/v${GITHUB_RUNNER_VERSION}/actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz \
    && tar xzf ./actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz && rm -f actions-runner-linux-x64-${GITHUB_RUNNER_VERSION}.tar.gz \
    && sed -i '3,9d' ./config.sh \
    && sed -i '3,8d' ./run-helper.sh.template

FROM ubuntu:20.04 as runner

ARG DEBIAN_FRONTEND=noninteractive
ENV KMS_SERVER_ADDR ""
ENV RUNNER_REGISTER_TO ""
ENV RUNNER_WORKDIR "_work"
ENV RUNNER_LABELS ""
ENV ADDITIONAL_PACKAGES ""
ENV ADDITIONAL_FLAGS ""

# Install base packages
RUN apt-get update \
    && apt-get install -y \
        curl \
        sudo \
        jq \
        iputils-ping \
        zip \
        libssl-dev \
        libcurl4-gnutls-dev \
        zlib1g-dev \
        gettext \
        make \
        build-essential \
        python3-pip \
        wget \
        cmake \
        clang \
        perl \
        psmisc \
        software-properties-common \
        openssh-client
# Install git
RUN wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.28.0.tar.gz \
    && tar -xvzf git-2.28.0.tar.gz \
    && cd git-2.28.0 \
    && ./configure --prefix=/usr/ \
    && make -j 8 \
    && make install \
    && cd && rm -rf git-2.28.0.tar.gz git-2.28.0
# Install docker
RUN curl https://download.docker.com/linux/static/stable/x86_64/docker-20.10.14.tgz --output docker-20.10.14.tgz \
    && tar xvfz docker-20.10.14.tgz \
    && mv docker/* /usr/bin/ \
    && rm -f docker-20.10.14.tgz
# Install nodejs
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash - \
    && sudo apt-get install -y nodejs
# Install puppeteer dependencies
RUN apt-get install -y \
    gconf-service \
    libgbm-dev \
    libasound2 \
    libatk1.0-0 \
    libc6 \
    libcairo2 \
    libcups2 \
    libdbus-1-3 \
    libexpat1 \
    libfontconfig1 \
    libgcc1 \
    libgconf-2-4 \
    libgdk-pixbuf2.0-0 \
    libglib2.0-0 \
    libgtk-3-0 \
    libnspr4 \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libstdc++6 \
    libx11-6 \
    libx11-xcb1 \
    libxcb1 \
    libxcomposite1 \
    libxcursor1 \
    libxdamage1 \
    libxext6 \
    libxfixes3 \
    libxi6 \
    libxrandr2 \
    libxrender1 \
    libxss1 \
    libxtst6 \
    ca-certificates \
    fonts-liberation \
    libappindicator1 \
    libnss3 \
    lsb-release \
    xdg-utils \
    wget

# Cean APT
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER root
WORKDIR /root/

COPY --from=env /root/ /root/
RUN  /root/bin/installdependencies.sh

COPY entrypoint.sh runsvc.sh ./
RUN sudo chmod u+x ./entrypoint.sh ./runsvc.sh

RUN install -m 600 -D /dev/null ~/.ssh/known_hosts
RUN ssh-keyscan -H github.com >> ~/.ssh/known_hosts

ENTRYPOINT ["./entrypoint.sh"]
