# Copyright VMware, Inc.
# SPDX-License-Identifier: APACHE-2.0

FROM buildpack-deps:bullseye as build

COPY prebuildfs /

ENV POSTGRESQL_VERSION 12.16
ENV POSTGRESQL_BASE_DIR "/opt/bitnami/postgresql"
ENV POSTGIS_VERSION 3.2.4

# Install PostgreSQL
RUN install_packages clang dirmngr gosu gnupg libclang-dev libicu-dev libipc-run-perl libkrb5-dev libldap2-dev liblz4-dev locales libpam-dev libperl-dev libpython3-dev libreadline-dev libssl-dev libxml2-dev libxslt1-dev llvm llvm-dev postgresql-server-dev-all python3-dev tcl-dev uuid-dev
RUN curl -sSL "https://ftp.postgresql.org/pub/source/v${POSTGRESQL_VERSION}/postgresql-${POSTGRESQL_VERSION}.tar.gz" | tar -xz && \
	cd "postgresql-${POSTGRESQL_VERSION}" && \
	./configure \
		--prefix="${POSTGRESQL_BASE_DIR}" \
		--enable-integer-datetimes \
		--enable-thread-safety \
		--enable-tap-tests \
		--with-uuid=e2fs \
		--with-gnu-ld \
		--with-pgport=5432 \
		--with-system-tzdata=/usr/share/zoneinfo \
		--with-includes=/usr/local/include \
		--with-libraries=/usr/local/lib \
		--with-krb5 \
		--with-gssapi \
		--with-ldap \
		--with-pam \
		--with-tcl \
		--with-perl \
		--with-python \
		--with-openssl \
		--with-libxml \
		--with-libxslt \
		--with-icu \
		--with-llvm \
		--with-lz4 && \
	make -j $(nproc) && \
	make install && \
	make clean
# Install PostGIS
RUN install_packages clang default-libmysqlclient-dev libclang-dev llvm llvm-dev libgdal-dev libgeos-dev libjson-c-dev libproj-dev libprotobuf-c-dev libxml2-dev protobuf-c-compiler
RUN	curl -sSL "https://download.osgeo.org/postgis/source/postgis-${POSTGIS_VERSION}.tar.gz" | tar -xz && \
	cd "postgis-${POSTGIS_VERSION}" && \
	./configure \
	    --prefix="${POSTGRESQL_BASE_DIR}" \
	    --with-pgconfig="${POSTGRESQL_BASE_DIR}/bin/pg_config" && \
	make -j $(nproc) && \
	make install && \
	make clean

RUN install_packages binutils
RUN find "${POSTGRESQL_BASE_DIR}" -type f -name '*.o' -o -perm /u=x,g=x,o=x -exec strip {} \; || true

FROM debian:11-slim

ARG EXTRA_LOCALES
ARG TARGETARCH
ARG WITH_ALL_LOCALES="no"

LABEL com.vmware.cp.artifact.flavor="sha256:1e1b4657a77f0d47e9220f0c37b9bf7802581b93214fff7d1bd2364c8bf22e8e" \
      org.opencontainers.image.base.name="docker.io/bitnami/minideb:bullseye" \
      org.opencontainers.image.created="2023-08-30T05:25:04Z" \
      org.opencontainers.image.description="Application packaged by VMware, Inc" \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.ref.name="12.16.0-debian-11-r17" \
      org.opencontainers.image.title="postgresql" \
      org.opencontainers.image.vendor="VMware, Inc." \
      org.opencontainers.image.version="12.16.0"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-11" \
    OS_NAME="linux"

COPY prebuildfs /
COPY --from=build /opt/bitnami/postgresql /opt/bitnami/postgresql
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates curl libbsd0 libbz2-1.0 libedit2 libffi7 libgcc-s1 libgmp10 libgnutls30 libhogweed6 libicu67 libidn2-0 libldap-2.4-2 liblz4-1 liblzma5 libmd0 libncurses6 libnettle8 libp11-kit0 libpcre3 libreadline8 libsasl2-2 libsqlite3-0 libssl1.1 libstdc++6 libtasn1-6 libtinfo6 libunistring2 libuuid1 libxml2 libxslt1.1 libzstd1 locales procps zlib1g
RUN install_packages libgeos-c1v5 libgdal28 libjson-c5 libproj19 libprotobuf-c1
RUN mkdir -p /tmp/bitnami/pkg/cache/ && cd /tmp/bitnami/pkg/cache/
RUN apt-get autoremove --purge -y curl && \
    apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami
RUN localedef -c -f UTF-8 -i en_US en_US.UTF-8
RUN update-locale LANG=C.UTF-8 LC_MESSAGES=POSIX && \
    DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales
RUN echo 'en_GB.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen
RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

COPY rootfs /
RUN /opt/bitnami/scripts/postgresql/postunpack.sh
RUN /opt/bitnami/scripts/locales/add-extra-locales.sh
ENV APP_VERSION="12.16.0" \
    BITNAMI_APP_NAME="postgresql" \
    LANG="en_US.UTF-8" \
    LANGUAGE="en_US:en" \
    NSS_WRAPPER_LIB="/opt/bitnami/common/lib/libnss_wrapper.so" \
    PATH="/opt/bitnami/postgresql/bin:$PATH"

RUN useradd -r -u 1001 -g root postgres

VOLUME [ "/bitnami/postgresql", "/docker-entrypoint-initdb.d", "/docker-entrypoint-preinitdb.d" ]

EXPOSE 5432

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/postgresql/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/postgresql/run.sh" ]
