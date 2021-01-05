FROM debian:buster-slim
MAINTAINER FlectraHQ <info@flectrahq.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        python3-num2words \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        xz-utils \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb \
    && echo 'ea8277df4297afc507c61122f3c349af142f31e5 wkhtmltox.deb' | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ buster-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Install Flectra
ENV FLECTRA_VERSION 2.0alpha
ARG FLECTRA_RELEASE=latest

RUN curl -o flectra.deb -sSL https://download.flectrahq.com/2.0/pub/deb/flectra_${FLECTRA_VERSION}.${FLECTRA_RELEASE}_all.deb \
    && curl -o flectra_${FLECTRA_VERSION}.${FLECTRA_RELEASE}_amd64.changes -sSL https://download.flectrahq.com/2.0/pub/deb/flectra_${FLECTRA_VERSION}.${FLECTRA_RELEASE}_amd64.changes \
    && echo "`sed -n '/^Checksums-Sha1:$/{n;p;n;p;n;p}' flectra_${FLECTRA_VERSION}.${FLECTRA_RELEASE}_amd64.changes | tail -1 | awk '{print $1}'` flectra.deb" | sha1sum -c \
    && apt-get update \
    && apt-get -y install --no-install-recommends ./flectra.deb \
    && rm -rf /var/lib/apt/lists/* flectra.deb

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./flectra.conf /etc/flectra/

# Set permissions and Mount /var/lib/flectra to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown flectra /etc/flectra/flectra.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R flectra /mnt/extra-addons
VOLUME ["/var/lib/flectra", "/mnt/extra-addons"]

# Expose Flectra services
EXPOSE 7073 7072 7071

# Set the default config file
ENV FLECTRA_RC /etc/flectra/flectra.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER flectra

ENTRYPOINT ["/entrypoint.sh"]
CMD ["flectra"]
