# syntax=docker/dockerfile:1

FROM registry.access.redhat.com/hi/core-runtime@sha256:47c4393878ee5848f91d9538dbe742b8cd04da6d1db80286c293460eeb5b1a6c
USER root

# renovate: datasource=github-releases depName=pi-hole/FTL
ARG FTL_VERSION=v6.6.2
# renovate: datasource=github-releases depName=pi-hole/pi-hole
ARG PIHOLE_CORE_VERSION=v6.4.2
# renovate: datasource=github-releases depName=pi-hole/web
ARG PIHOLE_WEB_VERSION=v6.5.1
# cronie is not in the Hummingbird repo; supercronic is a static binary drop-in
# designed for containers: reads crontab files directly, logs to stdout, handles SIGTERM
# renovate: datasource=github-releases depName=aptible/supercronic
ARG SUPERCRONIC_VERSION=v0.2.33

ENV DNSMASQ_USER=65532
ENV FTL_CMD=no-daemon

RUN --mount=type=bind,from=registry.access.redhat.com/hi/core-runtime:latest-builder,target=/builder \
    LD_LIBRARY_PATH=/builder/lib64:/builder/usr/lib64 \
    RPM_CONFIGDIR=/builder/usr/lib/rpm \
    /builder/usr/bin/dnf install -y \
    --installroot=/ \
    --setopt=reposdir=/builder/etc/yum.repos.d \
    --setopt=install_weak_deps=False \
    --setopt=tsflags=nodocs \
    bind-utils \
    binutils \
    curl \
    git \
    iproute \
    logrotate \
    procps-ng \
    psmisc \
    tzdata \
    unzip

RUN echo "pihole:x:65532:65532:Pi-hole:/:/sbin/nologin" >> /etc/passwd \
    && echo "pihole:x:65532:" >> /etc/group

ADD https://ftl.pi-hole.net/macvendor.db /macvendor.db

RUN git clone --branch ${PIHOLE_CORE_VERSION} --single-branch --depth 1 \
    https://github.com/pi-hole/pi-hole.git /etc/.pihole \
    && git clone --branch ${PIHOLE_WEB_VERSION} --single-branch --depth 1 \
    https://github.com/pi-hole/web.git /var/www/html/admin

RUN cd /etc/.pihole && \
    install -Dm755 -d /opt/pihole && \
    install -Dm755 -t /opt/pihole gravity.sh && \
    install -Dm755 -t /opt/pihole ./advanced/Scripts/*.sh && \
    install -Dm755 -t /opt/pihole ./advanced/Scripts/COL_TABLE && \
    install -Dm755 -d /etc/pihole && \
    install -Dm644 -t /etc/pihole ./advanced/Templates/logrotate && \
    install -Dm755 -d /var/log/pihole && \
    install -Dm755 -d /var/lib/logrotate && \
    install -Dm755 -t /usr/local/bin pihole && \
    install -Dm644 ./advanced/bash-completion/pihole.bash /etc/bash_completion.d/pihole && \
    install -Dm644 ./advanced/bash-completion/pihole-ftl.bash /etc/bash_completion.d/pihole-FTL && \
    install -T -m 0755 ./advanced/Templates/pihole-FTL-poststop.sh /opt/pihole/pihole-FTL-poststop.sh

COPY --chmod=0755 bash_functions.sh /usr/bin/bash_functions.sh
COPY --chmod=0755 prefix-time.sh /usr/bin/prefix-time.sh
COPY --chmod=0755 start.sh /usr/bin/start.sh
COPY --chmod=0755 pihole-FTL-prestart.sh /opt/pihole/pihole-FTL-prestart.sh
COPY crontab.txt /crontab.txt

RUN curl -sSL \
    "https://github.com/pi-hole/ftl/releases/download/${FTL_VERSION}/pihole-FTL-amd64" \
    -o /usr/bin/pihole-FTL \
    && chmod +x /usr/bin/pihole-FTL \
    && readelf -h /usr/bin/pihole-FTL || (echo "Error with FTL binary" && exit 1)

RUN curl -sSL \
    "https://github.com/aptible/supercronic/releases/download/${SUPERCRONIC_VERSION}/supercronic-linux-amd64" \
    -o /usr/bin/supercronic \
    && chmod +x /usr/bin/supercronic

EXPOSE 53 53/udp
EXPOSE 80

HEALTHCHECK CMD dig -p $(pihole-FTL --config dns.port) +short +norecurse +retry=0 @127.0.0.1 pi.hole || exit 1

ENTRYPOINT ["start.sh"]
