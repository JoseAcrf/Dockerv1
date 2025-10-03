# odoo/Dockerfile — build estable (evita apt 100 en Portainer)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# APT robusto (reintentos + IPv4)
RUN printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries

# Esenciales para compilar wheels de addons + runtimes mínimos (sin paquetes problemáticos)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    zlib1g-dev libjpeg62-turbo-dev libpng-dev \
    fontconfig libx11-6 libxext6 libxrender1 \
  ; rm -rf /var/lib/apt/lists/*

# Fuentes / xfonts (opcionales; si tu mirror no las tiene, NO rompe el build)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base xfonts-75dpi || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-core fonts-dejavu-extra fonts-liberation || true; \
  rm -rf /var/lib/apt/lists/*

# wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Debian 12
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in amd64|arm64) : ;; *) echo "unsupported arch: $ARCH"; exit 1;; esac; \
  curl -fsSL -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb"; \
  dpkg -i /tmp/wkhtmltox.deb || true; \
  apt-get update; \
  apt-get -f install -y --no-install-recommends; \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf; \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage; \
  rm -f /tmp/wkhtmltox.deb; \
  rm -rf /var/lib/apt/lists/*

# Requisitos pip de tus addons (rjsmin/rcssmin/psycogreen, etc.)
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
