FROM odoo:19.0

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash","-o","pipefail","-c"]
USER root
ARG WKHTML_VERSION=0.12.6.1-3

RUN set -eux; \
  printf 'Acquire::Retries "5";\nAcquire::ForceIPv4 "true";\n' > /etc/apt/apt.conf.d/99retries

RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl git build-essential python3-dev \
    libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
    fontconfig libx11-6 libxext6 libxrender1 libjpeg62-turbo libpng16-16 zlib1g \
  ; rm -rf /var/lib/apt/lists/*

RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends xfonts-base || true; \
  apt-get install -y --no-install-recommends xfonts-75dpi || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-core || true; \
  apt-get install -y --no-install-recommends fonts-dejavu-extra || true; \
  apt-get install -y --no-install-recommends fonts-liberation || true; \
  apt-get install -y --no-install-recommends fonts-liberation2 || true; \
  apt-get install -y --no-install-recommends gsfonts || true; \
  rm -rf /var/lib/apt/lists/*

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

COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
