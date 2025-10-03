# odoo/Dockerfile (fix apt-get exit code 100 by trimming to packages disponibles en Debian Bookworm)
FROM odoo:19.0

USER root
ARG WKHTML_VERSION=0.12.6.1-3

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates curl git build-essential python3-dev \
      libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
      fontconfig libx11-6 libxext6 libxrender1 libx11-xcb1 libxcb1 libxfixes3 \
      libxcursor1 libxdamage1 libxcomposite1 libxkbcommon0 libasound2 \
      libjpeg62-turbo libpng16-16 libfreetype6 liblcms2-2 libopenjp2-7 zlib1g \
      xfonts-75dpi xfonts-base \
      fonts-dejavu-core fonts-dejavu-extra fonts-liberation fonts-noto-cjk \
    ; \
    rm -rf /var/lib/apt/lists/*

# Instala wkhtmltopdf 0.12.6.1-3 (Qt parcheado) para Debian 12 (bookworm)
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

# (Opcional) instala pip de tus addons si existe requirements.txt
COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

# Logs
RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
