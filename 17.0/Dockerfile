FROM ubuntu:jammy
MAINTAINER Odoo S.A. <info@odoo.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG en_US.UTF-8

# Retrieve the target architecture to install the correct wkhtmltopdf package
ARG TARGETARCH

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf

COPY ./binaries/wkhtmlox.deb /opt/
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        python3-magic \
        python3-num2words \
        python3-odf \
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
	    fonts-inconsolata \
	    fonts-font-awesome \
	    fonts-roboto-unhinted \
	    gsfonts \
	    libjs-underscore \
	    lsb-base \
	    postgresql-client \
	    python3-babel \
	    python3-chardet \
	    python3-dateutil \
	    python3-decorator \
	    python3-docutils \
	    python3-freezegun \
	    python3-geoip2 \
	    python3-pil \
	    python3-jinja2 \
	    python3-libsass \
	    python3-lxml \
	    python3-num2words \
	    python3-ofxparse \
	    python3-passlib \
	    python3-polib \
	    python3-psutil \
	    python3-psycopg2 \
	    python3-pydot \
	    python3-openssl \
	    python3-pypdf2 \
	    python3-rjsmin \
	    python3-qrcode \
	    python3-reportlab \
	    python3-requests \
	    python3-stdnum \
	    python3-tz \
	    python3-vobject \
	    python3-werkzeug \
	    python3-xlsxwriter \
	    python3-zeep \
	    python3-ldap \
        xz-utils && \

    if [ -z "${TARGETARCH}" ]; then \
        TARGETARCH="$(dpkg --print-architecture)"; \
    fi; \
    WKHTMLTOPDF_ARCH=${TARGETARCH} && \
    case ${TARGETARCH} in \
    "amd64") WKHTMLTOPDF_ARCH=amd64 && WKHTMLTOPDF_SHA=967390a759707337b46d1c02452e2bb6b2dc6d59  ;; \
    "arm64")  WKHTMLTOPDF_SHA=90f6e69896d51ef77339d3f3a20f8582bdf496cc  ;; \
    "ppc64le" | "ppc64el") WKHTMLTOPDF_ARCH=ppc64el && WKHTMLTOPDF_SHA=5312d7d34a25b321282929df82e3574319aed25c  ;; \
    esac \
    && echo ${WKHTMLTOPDF_SHA} /opt/wkhtmltox.deb | sha1sum -c - \
    && apt-get install -y --no-install-recommends /opt/wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* /opt/wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ jammy-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
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

RUN adduser --system --home=/opt/odoo --group odoo

# Copy Odoo configuration file
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/opt/odoo", "/mnt/extra-addons", "/odoo", "/etc/odoo"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf
ENV PYTHONPATH="/odoo:$PYTHONPATH"

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

WORKDIR /odoo
CMD ["/odoo/odoo-bin"]
