---
aliases:
  - Deployment
tags:
  - dev/devops
  - dev/erp
date: 2026-05-04
---
**Related:** [[Odoo|Odoo]], [[Docker]], [[Compose]]

---

## Main idea

Deploy ``Odoo`` as _self-hosted_ using ``Docker`` to simulate production-ready usage.

---

## Goal

Being able to execute ``Odoo`` as _self-hosted_ "out of the box".

---

## Objectives

* User ``Docker`` to quickly create and easy manage ``Odoo`` deployment.
* Use ``Docker Compose`` to simulate a deployed ``PostgreSQL database`` alongside ``Odoo`` main app container.

---

## Details

### Workdir tree

```
odoo19/
├── base_addons/
│   ├── muk_theme/
│   ├── dark_theme/
│   └── ...
│
├── my_addons/
│   ├── estate_property/
│   ├── marketplace/
│   └── ...
│
├── odoo19/
│   ├── requirements.txt
│   └── ...
│
├── .dockerignore
├── docker-compose.yaml
├── Dockerfile
├── entrypoint.sh
├── odoo.conf
└── pyproject.toml
```


### .dockerignore

```plaintext
# Version control
.git
.gitignore

# Python
__pycache__
*.pyc
*.pyo
*.pyd
.Python
*.egg-info
dist/
build/

# Virtual environments
venv/
env/
.env
.venv

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Logs
*.log

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp

# Odoo local data (we don't want them in the image)
.local/
filestore/
sessions/

# Odoo source code (we copy it explicitly)
# odoo19/
```


### docker-compose.yml
```yml
services:

  # ─────────────────────────────────────────
  # PostgreSQL Database
  # ─────────────────────────────────────────
  db:
    image: postgres:18-alpine
    container_name: odoo19-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: odoo19
      POSTGRES_PASSWORD: openpgpwd
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - odoo19-db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - odoo19-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U odoo19 -d postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ─────────────────────────────────────────
  # Odoo 19
  # ─────────────────────────────────────────
  odoo:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: odoo19-app
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "8069:8069"   # Main web port
      - "8072:8072"   # Longpolling port (chat/notifications)
    environment:
      HOST: db
      PORT: 5432
      USER: odoo19
      PASSWORD: openpgpwd
    volumes:
      # Persistent Odoo data (filestore, sessions, etc.)
      - odoo19-data:/var/lib/odoo
      # Configuration (allows editing without rebuild)
      - ./odoo.conf:/etc/odoo/odoo.conf:ro
      # Custom addons in real time (hot-reload)
      - ./base_addons:/opt/odoo/base_addons
      - ./my_addons:/opt/odoo/my_addons
    networks:
      - odoo19-network
    command: ["odoo", "--dev=reload"]  # Remove --dev=reload in production

# ─────────────────────────────────────────
# Persistent volumes
# ─────────────────────────────────────────
volumes:
  odoo19-db-data:
    driver: local
  odoo19-data:
    driver: local

# ─────────────────────────────────────────
# Internal shared network
# ─────────────────────────────────────────
networks:
  odoo19-network:
    driver: bridge
```


### odoo19/requirements.txt

```
# The officially supported versions of the following packages are their
# python3-* equivalent distributed in Ubuntu 24.04 and Debian 12
asn1crypto==1.4.0 ; python_version < '3.11'
asn1crypto==1.5.1 ; python_version >= '3.11'
Babel==2.9.1 ; python_version < '3.11'  # min version = 2.6.0 (Focal with security backports)
Babel==2.10.3 ; python_version >= '3.11' and python_version < '3.13'
Babel==2.17.0 ; python_version >= '3.13'
cbor2==5.4.2 ; python_version < '3.12'
cbor2==5.6.2 ; python_version >= '3.12'
chardet==4.0.0 ; python_version < '3.11'  # (Jammy)
chardet==5.2.0 ; python_version >= '3.11'
cryptography==3.4.8; python_version < '3.12'  # incompatibility between pyopenssl 19.0.0 and cryptography>=37.0.0
cryptography==42.0.8 ; python_version >= '3.12'  # (Noble) min 41.0.7, pinning 42.0.8 for security fixes
docutils==0.17 ; python_version < '3.11'  # (Jammy)
docutils==0.20.1 ; python_version >= '3.11'
freezegun==1.1.0 ; python_version < '3.11'  # (Jammy)
freezegun==1.2.1 ; python_version >= '3.11' and python_version < '3.13'
freezegun==1.5.1 ; python_version >= '3.13'
geoip2==2.9.0
gevent==21.8.0 ; sys_platform != 'win32' and python_version == '3.10'  # (Jammy)
gevent==22.10.2; sys_platform != 'win32' and python_version > '3.10' and python_version < '3.12'
gevent==24.2.1 ; sys_platform != 'win32' and python_version >= '3.12' and python_version < '3.13'  # (Noble)
gevent==24.11.1 ; sys_platform != 'win32' and python_version >= '3.13'  # (Trixie)
greenlet==1.1.2 ; sys_platform != 'win32' and python_version == '3.10'  # (Jammy)
greenlet==2.0.2 ; sys_platform != 'win32' and python_version > '3.10' and python_version < '3.12'
greenlet==3.0.3 ; sys_platform != 'win32' and python_version >= '3.12' and python_version < '3.13' # (Noble)
greenlet==3.1.1 ; sys_platform != 'win32' and python_version >= '3.13'  # (Trixie)
idna==2.10 ; python_version < '3.12'  # requests 2.25.1 depends on idna<3 and >=2.5
idna==3.6 ; python_version >= '3.12'
Jinja2==3.0.3 ; python_version <= '3.10'
Jinja2==3.1.2 ; python_version > '3.10'
libsass==0.20.1 ; python_version < '3.11'
libsass==0.22.0 ; python_version >= '3.11'  # (Noble) Mostly to have a wheel package
lxml==4.8.0 ; python_version <= '3.10'
lxml==4.9.3 ; python_version > '3.10' and python_version < '3.12' # min 4.9.2, pinning 4.9.3 because of missing wheels for darwin in 4.9.3
lxml==5.2.1; python_version >= '3.12' # (Noble - removed html clean)
lxml-html-clean; python_version >= '3.12' # (Noble - removed from lxml, unpinned for futur security patches)
MarkupSafe==2.0.1 ; python_version <= '3.10'
MarkupSafe==2.1.2 ; python_version > '3.10' and python_version < '3.12'
MarkupSafe==2.1.5 ; python_version >= '3.12'  # (Noble) Mostly to have a wheel package
num2words==0.5.10 ; python_version < '3.12'  # (Jammy / Bookworm)
num2words==0.5.13 ; python_version >= '3.12'
ofxparse==0.21
openpyxl==3.0.9 ; python_version < '3.12'
openpyxl==3.1.2 ; python_version >= '3.12'
passlib==1.7.4 # min version = 1.7.2 (Focal with security backports)
Pillow==9.0.1 ; python_version <= '3.10'
Pillow==9.4.0 ; python_version > '3.10' and python_version < '3.12'
Pillow==10.2.0 ; python_version >= '3.12' and python_version < '3.13'  # (Noble) Mostly to have a wheel package
Pillow==11.1.0 ; python_version >= '3.13'  # (Noble) Mostly to have a wheel package
polib==1.1.1
psutil==5.9.0 ; python_version <= '3.10'
psutil==5.9.4 ; python_version > '3.10' and python_version < '3.12'
psutil==5.9.8 ; python_version >= '3.12' # (Noble) Mostly to have a wheel package
# psycopg2==2.9.2 ; python_version == '3.10' # (Jammy)
# psycopg2==2.9.5 ; python_version == '3.11'
# psycopg2==2.9.9 ; python_version >= '3.12' and python_version < '3.13' # (Noble)
# psycopg2==2.9.10 ; python_version >= '3.13' # (Trixie)
psycopg2-binary
pyopenssl==21.0.0 ; python_version < '3.12'
pyopenssl==24.1.0 ; python_version >= '3.12' # (Noble) min 23.2.0, pinned for compatibility with cryptography==42.0.8 and security patches
PyPDF2==1.26.0 ; python_version <= '3.10'
PyPDF2==2.12.1 ; python_version > '3.10' and python_version < '3.13' # (Noble and below)
PyPDF==5.4.0 ; python_version >= '3.13' # (Trixie)
pypiwin32 ; sys_platform == 'win32'
pyserial==3.5
python-dateutil==2.8.1 ; python_version < '3.11'
python-dateutil==2.8.2 ; python_version >= '3.11'
python-magic==0.4.24; sys_platform != 'win32' and python_version < '3.12'  # (jammy)
python-magic==0.4.27; sys_platform != 'win32' and python_version >= '3.12'  # (noble)
# python-ldap==3.4.0 ; sys_platform != 'win32' and python_version < '3.12' # min version = 3.2.0 (Focal with security backports)
# python-ldap==3.4.4 ; sys_platform != 'win32' and python_version >= '3.12'  # (Noble) Mostly to have a wheel package
python-ldap
python-stdnum==1.17 ; python_version < '3.11'  # (jammy)
python-stdnum==1.19 ; python_version >= '3.11'
pytz  # no version pinning to avoid OS perturbations
pyusb==1.2.1
qrcode==7.3.1 ; python_version < '3.11'  # (jammy)
qrcode==7.4.2 ; python_version >= '3.11'
reportlab==3.6.8 ; python_version <= '3.10'
reportlab==3.6.12 ; python_version > '3.10' and python_version < '3.12'
reportlab==4.1.0 ; python_version >= '3.12' # (Noble) Mostly to have a wheel package
requests==2.25.1 ;  python_version < '3.11' # versions < 2.25 aren't compatible w/ urllib3 1.26. Bullseye = 2.25.1. min version = 2.22.0 (Focal)
requests==2.31.0 ; python_version >= '3.11' # (Noble)
rjsmin==1.1.0 ; python_version < '3.11'  # (jammy)
rjsmin==1.2.0 ; python_version >= '3.11'
rl-renderPM==4.0.3 ; sys_platform == 'win32' and python_version >= '3.12'  # Needed by reportlab 4.1.0 but included in deb package
urllib3==1.26.5 ; python_version < '3.12' # indirect / min version = 1.25.8 (Focal with security backports)
urllib3==2.0.7  ; python_version >= '3.12'  # (Noble) Compatibility with cryptography
vobject==0.9.6.1
Werkzeug==2.0.2 ; python_version <= '3.10'
Werkzeug==2.2.2 ; python_version > '3.10' and python_version < '3.12'
Werkzeug==3.0.1 ; python_version >= '3.12'  # (Noble) Avoid deprecation warnings
xlrd==1.2.0 ; python_version < '3.12'  # (jammy)
xlrd==2.0.1 ; python_version >= '3.12'
XlsxWriter==3.0.2 ; python_version < '3.12'  # (jammy)
XlsxWriter==3.1.9 ; python_version >= '3.12'
xlwt==1.3.0
zeep==4.1.0 ; python_version < '3.11'  # (jammy)
zeep==4.2.1 ; python_version >= '3.11' and python_version < '3.13'
zeep==4.3.1 ; python_version >= '3.13'
```


### Dockerfile

```Dockerfile
FROM python:3.12-slim-bookworm

# Maintainer
LABEL maintainer="tu@email.com"

# Environment variables
ENV ODOO_VERSION=19.0 \
    ODOO_HOME=/opt/odoo \
    ODOO_DATA_DIR=/var/lib/odoo \
    LANG=C.UTF-8

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Build tools
    build-essential \
    # Base tools
    git \
    curl \
    wget \
    ca-certificates \
    gnupg \
    python3-dev \
    # Odoo dependencies
    libxml2-dev \
    libxslt1-dev \
    libldap2-dev \
    libsasl2-dev \
    libssl-dev \
    libjpeg-dev \
    libpng-dev \
    libpq-dev \
    zlib1g-dev \
    # node-less \
    npm \
    # Process tools
    gosu \
    # wkhtmltopdf for PDF reports
    wkhtmltopdf \
    # PostgreSQL client
    postgresql-client \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss for RTL support (Arabic, Hebrew, etc.)
RUN npm install -g rtlcss

# Create odoo user
RUN useradd -m -d ${ODOO_HOME} -U -r -s /bin/bash odoo

# Create necessary directories
RUN mkdir -p ${ODOO_DATA_DIR} \
    && mkdir -p ${ODOO_HOME}/custom-addons \
    && chown -R odoo:odoo ${ODOO_DATA_DIR} \
    && chown -R odoo:odoo ${ODOO_HOME}

# Copy Odoo 19 source code
# Assuming you have the odoo19 folder in the same directory as the Dockerfile
COPY --chown=odoo:odoo ./odoo19 ${ODOO_HOME}/odoo19

# Install Odoo Python dependencies
RUN pip install --no-cache-dir -r ${ODOO_HOME}/odoo19/requirements.txt

# Install additional useful Python dependencies
RUN pip install --no-cache-dir \
    psycopg2-binary \
    Pillow \
    gevent \
    inotify \
    watchdog

# Copy custom addons (if you have them)
COPY --chown=odoo:odoo ./base_addons ${ODOO_HOME}/base_addons
COPY --chown=odoo:odoo ./my_addons ${ODOO_HOME}/my_addons

# Copy Odoo configuration
COPY --chown=odoo:odoo ./odoo.conf /etc/odoo/odoo.conf

# Copy entrypoint script
COPY ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Expose Odoo HTTP port
EXPOSE 8069
# Longpolling port (real-time chat, notifications)
EXPOSE 8072

# Volume for persistent data
VOLUME ["${ODOO_DATA_DIR}"]

# Switch to odoo user
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
```


### entrypoint.sh

```bash
#!/bin/bash
set -e

# Wait for PostgreSQL to be ready (redundant with healthcheck, but just in case)
echo "Waiting for connection to PostgreSQL on ${HOST}:${PORT}..."
until pg_isready -h "${HOST}" -p "${PORT}" -U "${USER}" > /dev/null 2>&1; do
    echo "  PostgreSQL is not ready yet, retrying in 2s..."
    sleep 2
done
echo "PostgreSQL is ready."

# If the first argument is 'odoo', execute odoo-bin
if [ "$1" = "odoo" ]; then
    shift
    exec python /opt/odoo/odoo19/odoo-bin \
        --config=/etc/odoo/odoo.conf \
        "$@"
fi

# Any other command is executed directly
exec "$@"
```


### odoo.conf

```
[options]
; ── Server ──────────────────────────────
http_interface = 0.0.0.0
http_port = 8069
longpolling_port = 8072
# Set to True if behind a proxy (Nginx, Apache, etc.)
proxy_mode = False
# Maximum number of requests that can be processed per second
limit_request = 8196

; ── Database ─────────────────────────
db_host = db
db_port = 5432
db_user = odoo19
db_password = openpgpwd
db_maxconn = 64
# Tries SSL, but fallbacks to non-SSL if needed
db_sslmode = prefer
# Use template1 as default database template
db_template = template1
dbfilter = .*
# Use my_db as default database name. Empty = multidb mode
# db_name = my_db
# List all databases in the database list page
list_db = True

; ── Addons paths ───────────────────────
addons_path = /opt/odoo/odoo19/addons,/opt/odoo/odoo19/odoo/addons,/opt/odoo/base_addons,/opt/odoo/my_addons

; ── Data ─────────────────────────────────
data_dir = /var/lib/odoo

; ── SMPT server ─────────────────────────────────
# smtp_server = localhost
# smtp_port = 25
# smtp_ssl = False
# smtp_user = False
# smtp_password = False

; ── Workers (0 = threading mode, without gevent) ──
; In production change to: workers = 4
workers = 0
# Maximum number of threads for cron jobs
max_cron_threads = 2

; ── Time limits ─────────────────────
# If a worker exceeds this limit, it will be killed
limit_memory_hard = 2684354560
# If a worker exceeds this limit, it will be restarted when finished
limit_memory_soft = 2147483648
# Maximum CPU time in seconds
limit_time_cpu = 600
# Maximum real time in seconds
limit_time_real = 1200
# Maximum real time in seconds for cron jobs
# limit_time_real_cron = 600

; ── Log ───────────────────────────────────
log_level = info
# :CRITICAL :ERROR :WARNING :DEBUG
log_handler = :INFO
# log_file = /var/log/odoo/odoo.log
log_db = False
log_db_level = warning

; ── Behavior ─────────────────────────────
server_wide_modules = base,web
# If true, activates searches without accents (requires postgress unaccent extension)
unaccent = False
# If true, prevents the demo data from being loaded
without_demo = False
# If true, enables test mode (must be False in production)
test_enable = False

; ── Security ─────────────────────────────
admin_passwd = controlSebas

```


### pyproject.toml

```toml
[tool.ruff]
line-length = 79
target-version = "py311"

[tool.ruff.lint]
select = [
	"E",   # Basic errors
	"F",   # Basic errors
	"I",   # Imports
	"UP",  # Modernize syntax
	"B",   # Bugbear
	"N",   # Naming
	"C90", # Cyclomatic complexity
	"ANN", # Type annotations
	"S",   # Security
]
ignore = [
	"S311",  # Predictable random numbers
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.ruff.lint.isort]
known-first-party = ["my_addons", "base_addons", "odoo19"]

[tool.pylint.messages_control]
ignored-modules = ["odoo"]

[tool.ruff.lint.per-file-ignores]
"**/__manifest__.py" = [
	"F401",  # Unused import
	"F821",  # Undefined name
	"B018",  # Unused variable
	"E402",  # Module level import not at top of file
]
```

---

## Claude Sessions
