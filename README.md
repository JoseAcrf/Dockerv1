# Odoo 19 CE - Stack de producción (Docker Compose)

Incluye:
- Odoo 19 CE con `wkhtmltopdf 0.12.6.1-3 (Qt parcheado)`
- PostgreSQL 16 con extensiones `unaccent` y `pg_trgm`
- Dependencias extra (fonts, libs de imagen) para PDFs multi-idioma y render estable
- `rcssmin`/`rjsmin` (minificado JS/CSS) y `psycogreen` para workers
- Ejemplo de `nginx.conf` para proxy, compresión y caché de assets

## Estructura
```
.
├─ docker-compose.yml
├─ addons/              # tus módulos personalizados (vacío)
├─ config/
│  └─ odoo.conf         # configuración Odoo (edita admin_passwd)
├─ db/
│  └─ init/
│     └─ 01-extensions.sql  # crea extensiones en DB inicial y template1
├─ logs/                # logs de Odoo
├─ odoo/
│  ├─ Dockerfile
│  └─ requirements.txt  # pip para addons (rjsmin/rcssmin/psycogreen incl.)
└─ proxy/
   └─ nginx.conf        # ejemplo de proxy reverso
```

## Uso rápido (Portainer o CLI)
1. Edita `config/odoo.conf` y cambia `admin_passwd = CAMBIA_ESTA_CLAVE`.
2. (Opcional) Añade requirements de tus addons a `odoo/requirements.txt`.
3. Lanza el stack:
   ```bash
   docker compose build
   docker compose up -d
   ```
4. Accede a `http://TU_IP:8069`.
5. (Opcional) Levanta Nginx como contenedor o en otro host, usando `proxy/nginx.conf` y TLS.

## Comprobaciones
```bash
docker exec -it odoo19 wkhtmltopdf --version
# Debe mostrar: 0.12.6.1-3 (with patched qt)

# Prueba de PDF
docker exec -it odoo19 sh -c "echo '<h1>PDF OK</h1>' > /tmp/t.html && wkhtmltopdf /tmp/t.html /tmp/t.pdf && ls -lh /tmp/t.pdf"
```

## Notas
- El contenedor Odoo usa `/etc/odoo/odoo.conf`, `/mnt/extra-addons`, `/var/lib/odoo` y `/var/log/odoo` con volúmenes para persistencia.
- El script `db/init/01-extensions.sql` asegura que **todas** las nuevas BDs tengan `unaccent`/`pg_trgm`.
- Si tus PDFs muestran cuadrados o caracteres faltantes, añade más fuentes Noto según idioma.

¡Éxitos!
