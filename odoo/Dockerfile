# odoo/Dockerfile (minimal y alineado con lo imprescindible)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# APT robusto (reintentos + IPv4)
RUN printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries

# Solo lo necesario para Odoo 19 + compilar wheels de addons
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libsasl2-dev libldap2-dev libffi-dev libssl-dev \
    libxml2-dev libxslt1-dev zlib1g-dev \
    libjpeg62-turbo-dev libpng-dev liblcms2-dev \
    fontconfig libx11-6 libxext6 libxrender1 xfonts-base xfonts-75dpi \
  ; rm -rf /var/lib/apt/lists/*

# wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Debian 12 (Bookworm)
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  curl -fsSL -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${ARCH}.deb"; \
  dpkg -i /tmp/wkhtmltox.deb || true; \
  apt-get update; \
  apt-get -f install -y --no-install-recommends; \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf; \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage; \
  rm -f /tmp/wkhtmltox.deb; \
  rm -rf /var/lib/apt/lists/*

# Requisitos pip de addons (si existe)
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
