# Nginx Reverse Proxy Configuration — lld-infrastructure

Zentrale Konfiguration des Nginx Reverse Proxy (öffentlicher Einstiegspunkt).

## Einbindung in Docker Compose

Die Konfigurationsdateien werden von `proxy.yml` eingebunden:

```yaml
volumes:
  - ./../../docker/nginx:/etc/nginx/conf.d
```

**Pfad:** `docker/nginx/` → `Docker Container /etc/nginx/conf.d`

## Dateien

### default.conf (Aktuell)

**Status:** Production-Ready

Zentrale Reverse Proxy Konfiguration mit:

- **Health-Check Endpoint** (`/health`)
  - Used by Docker Healthcheck
  - Returns: 200 OK
  - No logging (performance)

- **Main Routing** (`/`)
  - Proxy-Pass zu Website Service: `http://website/`
  - Header-Propagation:
    - Host: Original Host
    - X-Real-IP: Client IP
    - X-Forwarded-For: Proxy Chain
    - X-Forwarded-Proto: http/https
  - Connection Settings:
    - HTTP/1.1 (Connection Reuse)
    - Timeouts: connect=60s, send=60s, read=60s

- **Dokumentation**
  - Zukünftige Erweiterungen beschrieben
  - Sprint 7: SSL/HTTPS, Domain-Routing
  - Sprint 8+: Multi-Service Routing

**Performance:**
- ✅ HTTP/1.1 Connection Reuse
- ✅ Reasonable Timeouts
- ✅ Header Forwarding (für Applications)

### Zukünftige Dateien (Sprint 7+)

```
docker/nginx/
├── default.conf         ← Health + Fallback
├── panel.conf           ← Virtual Host: panel.lukasleimer.dev
├── website.conf         ← Virtual Host: lukasleimer.dev
├── customers.conf       ← Virtual Hosts: customer-*.lukasleimer.dev
├── ssl.conf             ← SSL/TLS Konfiguration
└── README.md            ← Diese Datei
```

## Server Block Pattern

```nginx
# Template für zukünftige Virtual Hosts (Sprint 7)
server {
    listen 80;
    listen [::]:80;
    server_name panel.lukasleimer.dev;
    
    location / {
        proxy_pass http://panel/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

## Reload ohne Neustart (Entwicklung)

```bash
# Konfiguration validieren
docker exec lld-nginx nginx -t

# Reload (ohne Downtime)
docker exec lld-nginx nginx -s reload

# Logs anschauen
docker compose logs -f nginx
```

## Testing

### Health-Check
```bash
curl http://localhost/health
# Expected: 200 OK
```

### Request Flow
```bash
curl http://localhost/
# Expected: 200 OK + Website HTML
```

### Proxy Headers (Debug)
```bash
curl -v http://localhost/
# Check: X-Real-IP, X-Forwarded-For gesetzt
```

### Aus anderem Container
```bash
docker exec -it lld-nginx wget -O - http://website/
# Expected: Website HTML
```

## Konfigurationsstandards

### Sichere Header

Sollten in allen Server Blocks definiert sein:
```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "1; mode=block" always;
```

### Proxy Headers

Sollten in allen proxy_pass Locations definiert sein:
```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_http_version 1.1;
proxy_set_header Connection "";
```

### Timeouts

Sollten produktionsnah sein:
```nginx
proxy_connect_timeout 60s;    # Connection Establishment
proxy_send_timeout 60s;        # Request Sending
proxy_read_timeout 60s;        # Response Reading
```

## Performance-Tuning (Zukünftig)

```nginx
# Upstream Services
upstream panel_backend {
    server panel:8000;
    keepalive 32;
}

# Caching (später)
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=cache:10m;

# Rate Limiting (später)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/s;
```

## SSL/HTTPS (Sprint 7)

```nginx
# HTTP → HTTPS Redirect
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}

# HTTPS Listener
server {
    listen 443 ssl http2;
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    # ...
}
```

## Debugging

### Config Validierung
```bash
docker exec lld-nginx nginx -t
```

### View Config
```bash
docker exec lld-nginx cat /etc/nginx/nginx.conf
```

### Access Logs
```bash
docker compose logs nginx
# oder
docker exec lld-nginx tail -f /var/log/nginx/access.log
```

### Error Logs
```bash
docker exec lld-nginx tail -f /var/log/nginx/error.log
```

## Sicherheit

- ✅ Kein Root User (Nginx Standard: www-data)
- ✅ Alpine Image (minimale Oberfläche)
- ✅ Security Headers
- ✅ Versioned Image (kein latest)
- ⏳ SSL/TLS (Sprint 7)
- ⏳ Rate Limiting (Sprint 8)
- ⏳ WAF Rules (Future)

## Architektur-Vorgaben

- **Single Point of Entry:** Nginx ist der einzige öffentliche Port
- **Zero Trust Networking:** Internal Services nicht direkt erreichbar
- **Configuration as Code:** Alles in Git versioniert
- **Infrastructure as Code:** Docker-basiert, reproduzierbar
- **Production Ready:** Linting, Validation, Error Handling

