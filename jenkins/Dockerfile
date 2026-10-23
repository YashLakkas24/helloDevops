FROM jenkins/jenkins:lts-jdk21

USER root

# Install Git and the Docker CLI. The Docker daemon runs in a separate
# Docker-in-Docker service defined in compose.yaml.
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl git \
    && install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" > /etc/apt/sources.list.d/docker.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends docker-ce-cli \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Jenkins manages the toolchain and pipeline plugins.
USER jenkins
RUN jenkins-plugin-cli --plugins "git workflow-aggregator docker-workflow"
