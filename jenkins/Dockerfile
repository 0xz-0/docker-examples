FROM jenkins/jenkins:2.568.3-lts-jdk21

USER root

ARG DOCKER_CLI_VERSION=29.8.1
ARG DOCKER_CLI_SHA256=d8db66739d2e28d4933786d73e918d9be643a67fbd835db1bf740d650a259e70

RUN curl -fsSL "https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_CLI_VERSION}.tgz" -o /tmp/docker-cli.tgz \
    && echo "${DOCKER_CLI_SHA256}  /tmp/docker-cli.tgz" | sha256sum -c - \
    && tar -xzf /tmp/docker-cli.tgz -C /tmp \
    && mv /tmp/docker/docker /usr/local/bin/docker \
    && rm -rf /tmp/docker-cli.tgz /tmp/docker

USER jenkins
