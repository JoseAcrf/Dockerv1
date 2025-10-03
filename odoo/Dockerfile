# odoo/Dockerfile — robusto para Bookworm y transición t64 (evita apt exit code 100)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Endurece APT y normaliza mirrors
RUN set -eux; \
  printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries; \
  sed -ri 's#http(s)?://[^ ]*security\.debian\.org/updates#http://security.debian.org/debian-security#g' /etc/apt/sources.list || true

# Paquetes base seguros en Debian 12 (sin t64)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    fontconfig xfonts-base xfonts-75dpi \
    fonts-dejavu-core fonts-dejavu-extra fonts-liberation2 gsfonts \
  ; rm -rf /var/lib/apt/lists/*

# Libs X/imagen con fallback a sufijo t64 cuando aplique
RUN set -eux; \
  apt-get update; \
  (apt-get install -y --no-install-recommends \
      libx11-6 libxext6 libxrender1 libjpeg62-turbo libpng16-16 \
  ) || (apt-get install -y --no-install-recommends \
      libx11-6t64 libxext6t64 libxrender1t64 libjpeg62-turbo libpng16-16t64 \
  ); \
  rm -rf /var/lib/apt/lists/*

# wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Bookworm
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in amd64|arm64) : ;; *) echo "unsupported arch: $ARCH"; exit 1;; esac; \
  curl -fsSL -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${ARCH}.deb"; \
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
