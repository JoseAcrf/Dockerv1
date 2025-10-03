# odoo/Dockerfile (robusto frente a "exit code 100" en apt)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Paquetes base (estrictamente necesarios)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    fontconfig libxrender1 libxext6 libx11-6 libjpeg62-turbo libpng16-16 zlib1g \
  ; rm -rf /var/lib/apt/lists/*

# Paquetes opcionales (si fallan en tu mirror, el build continua)
RUN set -eux; apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base xfonts-75dpi || true; \
  rm -rf /var/lib/apt/lists/*

RUN set -eux; apt-get update; \
  apt-get install -y --no-install-recommends fonts-dejavu-core fonts-dejavu-extra fonts-liberation || true; \
  rm -rf /var/lib/apt/lists/*

RUN set -eux; apt-get update; \
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
RUN if [ -s /opt/odoo/requirements.txt ] ; then pip install --no-cache-dir -r /opt/odoo/requirements.txt ; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
