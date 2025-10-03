# Dockerfile.custom — basado en tu referencia, actualizado a Odoo 19 e instala wkhtmltopdf (lector PDF)
FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

# Paquetes Python extra (según tu referencia)
RUN pip install --no-cache-dir pandas numpy requests beautifulsoup4

# Dependencias de sistema mínimas (según tu referencia, +curl/ca-certificates)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    python3-dev libxml2-dev libxslt1-dev ca-certificates curl \
  ; rm -rf /var/lib/apt/lists/*

# Instala wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Debian 12 (Bookworm)
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

# Fuentes/xfonts opcionales para PDFs (no rompe el build si el mirror no las tiene)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base xfonts-75dpi || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-core fonts-dejavu-extra fonts-liberation || true; \
  rm -rf /var/lib/apt/lists/*

USER odoo
