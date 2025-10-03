# odoo/Dockerfile
FROM odoo:19.0

USER root
ARG WKHTML_VERSION=0.12.6.1-3

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates curl git build-essential python3-dev \
      libpq-dev libldap2-dev libsasl2-dev libffi-dev libxml2-dev libxslt1-dev \
      fontconfig libx11-6 libxext6 libxrender1 libx11-xcb1 libxcb1 libxfixes3 \
      libxcursor1 libxdamage1 libxcomposite1 libxkbcommon0 libasound2 \
      libjpeg62-turbo libpng16-16 libpango-1.0-0 libharfbuzz0b libglib2.0-0 \
      xfonts-75dpi xfonts-base \
      fonts-dejavu-core fonts-dejavu-extra fonts-liberation fonts-freefont-ttf fonts-noto-cjk \
      fonts-noto-color-emoji \
      libfreetype6 liblcms2-2 libwebp7 libtiff6 libopenjp2-7 zlib1g \
    && rm -rf /var/lib/apt/lists/*

RUN ARCH="$(dpkg --print-architecture)"; \
    case "$ARCH" in \
      amd64) DEB_ARCH="amd64" ;; \
      arm64) DEB_ARCH="arm64" ;; \
      *) echo "Arquitectura no soportada: $ARCH" && exit 1 ;; \
    esac && \
    curl -L -o /tmp/wkhtmltox.deb \
      "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTML_VERSION}/wkhtmltox_${WKHTML_VERSION}.bookworm_${DEB_ARCH}.deb" && \
    apt-get update && apt-get install -y --no-install-recommends /tmp/wkhtmltox.deb && \
    ln -sf /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf && \
    ln -sf /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage && \
    rm -f /tmp/wkhtmltox.deb

COPY requirements.txt /opt/odoo/requirements.txt
RUN if [ -s /opt/odoo/requirements.txt ]; then pip install --no-cache-dir -r /opt/odoo/requirements.txt; fi

RUN mkdir -p /var/log/odoo && chown -R odoo:odoo /var/log/odoo
USER odoo
