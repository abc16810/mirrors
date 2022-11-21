FROM docker.io/calico/typha:master

ARG UBI_IMAGE=registry.access.redhat.com/ubi8/ubi-minimal:8.6
ARG GIT_VERSION=unknown

FROM ${UBI_IMAGE} as ubi

# Since our binary isn't designed to run as PID 1, run it via the tini init daemon.
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static-amd64 /sbin/tini
RUN chmod +x /sbin/tini

# Add in top-level license file
RUN mkdir /licenses

FROM scratch
ARG GIT_VERSION
LABEL name="Calico Typha" \
      vendor="Project Calico" \
      version=$GIT_VERSION \
      release="1" \
      summary="Calico Typha is a fan-out datastore proxy" \
      description="Calico Typha is a fan-out datastore proxy" \
      maintainer="Shaun Crampton <shaun@tigera.io>"

COPY --from=ubi /sbin/tini /sbin/tini
COPY --from=ubi /licenses /licenses

COPY --from=ubi /usr/include /usr/include

# lib dependencies for typha
COPY --from=ubi /lib64/libpthread.so.0 /lib64/libpthread.so.0
COPY --from=ubi /lib64/libc.so.6 /lib64/libc.so.6
COPY --from=ubi /lib64/ld-linux-x86-64.so.2 /lib64/ld-linux-x86-64.so.2
COPY --from=ubi /lib64/libnss_dns.so.2 /lib64/libnss_dns.so.2
COPY --from=ubi /lib64/libresolv.so.2 /lib64/libresolv.so.2

# Copy hostname configuration files from the UBI image so glibc hostname lookups work.
COPY --from=ubi /etc/host.conf /etc/host.conf
COPY --from=ubi /etc/nsswitch.conf /etc/nsswitch.conf

# Put our binary in /code rather than directly in /usr/bin.  This allows the downstream builds
# to more easily extract the build artefacts from the container.
ADD bin/calico-typha-amd64 /code/calico-typha
ADD typha.cfg /etc/calico/typha.cfg

# Add nsswitch.conf so that we correctly resolve localhost based on /etc/hosts.
ADD nsswitch.conf /etc/nsswitch.conf

WORKDIR /code
ENV PATH="$PATH:/code"

USER 999
# Run Typha by default
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["calico-typha"]
