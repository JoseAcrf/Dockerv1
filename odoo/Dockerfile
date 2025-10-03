# odoo/Dockerfile — ajustado para compilar rcssmin/rjsmin/psycogreen
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# APT básico (evitamos listas largas problemáticas)
RUN set -eux; \
  printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries; \
  apt-get update; \
  apt-get install -y --no-install-recommends ca-certificates curl; \
  rm -rf /var/lib/apt/lists/*

# wkhtmltopdf 0.12.6.1-3 (Qt parcheado) y dependencias resueltas automáticamente
RUN set -eux; \
  ARCH="$(dpkg --print-architecture)"; \
  case "$ARCH" in amd64|arm64) : ;; *) echo "unsupported arch: $ARCH"; exit 1;; esac; \
  curl -fsSL -o /tmp/wkhtmltox.deb \
    "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${ARCH}.deb"; \
  dpkg -i /tmp/wkhtmltox.deb || true; \
  apt-get update; \
  apt-get -f install -y --no-install-recommends; \
  ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf; \
  ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage; \
  rm -f /tmp/wkhtmltox.deb; \
  rm -rf /var/lib/apt/lists/*

# (Opcional) fuentes/xfonts — si el mirror no las tiene, no rompe el build
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base xfonts-75dpi || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-core fonts-dejavu-extra fonts-liberation || true; \
  rm -rf /var/lib/apt/lists/*

# Requisitos pip de addons: instala deps de compilación temporalmente y luego purga
COPY requirements.txt /opt/odoo/requirements.txt
RUN set -eux; \
  if [ -s /opt/odoo/requirements.txt ]; then \
    apt-get update; \
    apt-get install -y --no-install-recommends build-essential python3-dev; \
    pip install --no-cache-dir -r /opt/odoo/requirements.txt; \
    apt-get purge -y build-essential python3-dev; \
    apt-get autoremove -y; \
    rm -rf /var/lib/apt/lists/*; \
  fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
