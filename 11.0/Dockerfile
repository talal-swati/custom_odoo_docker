FROM ubuntu:bionic
MAINTAINER Odoo S.A. <info@odoo.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends tzdata \
    && rm -rf /var/lib/apt/lists/*
RUN ln -fs /usr/share/zoneinfo/Asia/Jakarta /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            dirmngr \
            fonts-noto-cjk \
            gnupg \
            nano libjpeg-dev zlib1g-dev libpq-dev libmysqlclient-dev sudo git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less \
        && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.bionic_amd64.deb \
        && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb
RUN pip3 install wheel
RUN pip3 install websocket
RUN pip3 install websocket-client
RUN pip3 install tornado
# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
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
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - \
    && apt-get install --no-install-recommends -y nodejs \
    && npm install -g --force less less-plugin-clean-css\
    && rm -rf /var/lib/apt/lists/*

# Install Odoo
COPY ./odoo.deb /opt/
RUN apt-get update \
        && apt-get -y install --no-install-recommends /opt/odoo.deb \
        && rm -rf /var/lib/apt/lists/* odoo.deb


# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/
COPY ./bsp_req.txt /etc/odoo/

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons \
    && chown -R odoo /etc/odoo 
VOLUME ["/var/lib/odoo", "/mnt/extra-addons","/etc/odoo"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py
RUN pip3 install -r /etc/odoo/bsp_req.txt
RUN pip3 install git+https://github.com/aeroo/aeroolib.git
RUN pip3 install git+https://github.com/aeroo/currency2text.git
# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
