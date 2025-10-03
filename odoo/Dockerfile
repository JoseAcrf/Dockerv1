# odoo/Dockerfile — minimal (evita apt exit code 100)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# APT robusto y solo paquetes esenciales (evitamos listas largas problemáticas)
RUN set -eux; \
  printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries; \
  apt-get update; \
  apt-get install -y --no-install-recommends ca-certificates curl; \
  rm -rf /var/lib/apt/lists/*

# Instala wkhtmltopdf 0.12.6.1-3 (Qt parcheado) y que APT resuelva dependencias
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

# Fuentes/xfonts OPCIONALES (no rompen el build si tu mirror no las tiene)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base xfonts-75dpi || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-core fonts-dejavu-extra fonts-liberation || true; \
  rm -rf /var/lib/apt/lists/*

# Requisitos pip de addons (si existe el archivo)
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
