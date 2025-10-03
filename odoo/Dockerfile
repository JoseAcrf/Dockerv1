# odoo/Dockerfile (paqueteado mínimo y compatible para evitar exit code 100)
FROM odoo:19.0

USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Paquetes base + toolchain para compilar wheels (rjsmin/rcssmin/psycogreen) y libs necesarias
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git \
    build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    fontconfig libx11-6 libxext6 libxrender1 \
    xfonts-75dpi xfonts-base \
    libjpeg62-turbo libpng16-16 \
    fonts-dejavu-core fonts-dejavu-extra fonts-liberation \
  ; rm -rf /var/lib/apt/lists/*

# Fuentes CJK (opcional); si no están en mirrors del entorno, no falla el build
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends fonts-noto-cjk || true; \
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
  apt-get update; \
  apt-get install -y --no-install-recommends /tmp/wkhtmltox.deb; \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf; \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage; \
  rm -f /tmp/wkhtmltox.deb; \
  rm -rf /var/lib/apt/lists/*

# Requisitos pip de addons (rjsmin/rcssmin/psycogreen, etc.)
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
