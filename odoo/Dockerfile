# odoo/Dockerfile (maneja mirrors inestables y transición t64 de Debian)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Endurece APT (reintentos, IPv4) y normaliza mirrors
RUN set -eux; \
  printf 'Acquire::Retries "5";\nAcquire::http::Pipeline-Depth "0";\nAcquire::http::No-Cache "true";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries; \
  sed -ri 's#http(s)?://[^ ]*debian\.org/debian#http://deb.debian.org/debian#g' /etc/apt/sources.list || true; \
  sed -ri 's#http(s)?://[^ ]*security\.debian\.org/updates#http://security.debian.org/debian-security#g' /etc/apt/sources.list || true

# Paquetes base (si falla por nombres t64, probamos alternativas)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev zlib1g \
    fontconfig xfonts-base xfonts-75dpi \
    fonts-dejavu-core fonts-dejavu-extra fonts-liberation \
  ; \
  (apt-get install -y --no-install-recommends \
      libx11-6 libxext6 libxrender1 libjpeg62-turbo libpng16-16 \
   ) || (apt-get install -y --no-install-recommends \
      libx11-6t64 libxext6t64 libxrender1t64 libjpeg62-turbo libpng16-16t64 \
   ); \
  rm -rf /var/lib/apt/lists/*

# wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Debian 12 (bookworm)
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in \
    amd64) DEB_ARCH="amd64" ;; \
    arm64) DEB_ARCH="arm64" ;; \
    *) echo "Arquitectura no soportada: $ARCH" && exit 1 ;; \
  esac; \
  curl -L -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${DEB_ARCH}.deb"; \
  dpkg -x /tmp/wkhtmltox.deb /; \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf; \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage; \
  rm -f /tmp/wkhtmltox.deb

# Requisitos pip de addons (rjsmin/rcssmin/psycogreen, etc.)
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
