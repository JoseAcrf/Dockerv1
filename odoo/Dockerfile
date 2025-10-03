FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Configuración de APT para mejorar estabilidad
RUN echo 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";' > /etc/apt/apt.conf.d/99retries

# Instalación de dependencias principales
RUN set -eux; \
  apt-get update && \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    fontconfig libx11-6 libxext6 libxrender1 libjpeg-dev libpng-dev zlib1g && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Instalación de fuentes necesarias para wkhtmltopdf
RUN set -eux; \
  apt-get update && \
  apt-get install -y --no-install-recommends \
    xfonts-base xfonts-75dpi \
    fonts-dejavu-core fonts-dejavu-extra \
    fonts-liberation fonts-liberation2 \
    gsfonts && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Instalación de wkhtmltopdf
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in amd64|arm64) : ;; *) echo "unsupported arch: $ARCH"; exit 1;; esac; \
  curl -fsSL -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${ARCH}.deb"; \
  apt-get update && \
  apt-get install -y --no-install-recommends /tmp/wkhtmltox.deb && \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf && \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage && \
  rm -f /tmp/wkhtmltox.deb && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Instalación de dependencias Python si existen
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Preparación de logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo

USER odoo
