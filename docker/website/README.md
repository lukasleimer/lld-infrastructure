# Website Service — lld-infrastructure

Test-Website für Infrastruktur-Validierung.

## Verzeichnisstruktur

```
docker/website/
├── html/                    ← Statische HTML-Inhalte
│   └── index.html          ← Test-Webseite
├── default.conf            ← Nginx-Konfiguration
├── README.md               ← Diese Datei
```

## Dateien

### html/index.html

Statische Test-Webseite mit:
- Professional HTML5 Layout
- CSS Gradient Design
- Service Information (Namen, Status, Network)
- Status: Running ✓
- Purpose: End-to-End Testing

**Mount im Container:** `/usr/share/nginx/html/index.html`

**Zugriffsrechte:** Read-Only (ro) — keine Änderungen aus Container

### default.conf

Nginx-Konfiguration für statische Dateien:
- Server Block auf Port 80
- Root: `/usr/share/nginx/html`
- Index: `index.html`, `index.htm`
- Gzip Kompression: Delegiert zu globaler Nginx-Config (keine Duplikate)
- Cache-Strategien:
  - Assets (css, js): 7 Tage
  - Bilder (jpg, png, etc.): 30 Tage
  - HTML: No-Cache (für schnelle Updates)
- Security Headers:
  - X-Content-Type-Options: nosniff
  - X-Frame-Options: DENY
  - X-XSS-Protection: active
  - Referrer-Policy: strict-origin-when-cross-origin
- Error Handling: 404 und 50x Error Pages

**Mount im Container:** `/etc/nginx/conf.d/default.conf`

**Zugriffsrechte:** Read-Only (ro)

## Testing

### Lokal auf dem Host

```bash
# Test über Reverse Proxy (Port 80)
curl http://localhost/

# Browser
open http://localhost  # MacOS
start http://localhost  # Windows
```

### Aus anderem Container (im lld Network)

```bash
docker exec -it lld-nginx sh
wget -O - http://website/
```

### Docker Compose Healthcheck

```bash
docker compose ps  # Website Status

# Expected Output:
# lld-website    nginx:1.26-alpine    Up (healthy)
```

## Produktionsaspekte

✅ **Implementiert:**
- Alpine-basiertes Image (leicht, sicher)
- Versioned Image (kein latest)
- Read-Only Mounts (Sicherheit)
- Healthcheck (Überwachung)
- Auto-Restart (Zuverlässigkeit)
- Security Headers (Best Practice)
- Cache-Kontrolle (Performance)

⏳ **Geplant (Sprint 7+):**
- SSL/HTTPS Support
- Domain-Routing
- Custom Application Framework
- Echte Website-Anwendung

## Zukünftige Migration

Dieser Service wird später durch echte Anwendung ersetzt:

```yaml
# Beispiel (zukünftig):
services:
  website:
    image: our-registry/lld-website:1.0
    # ... statt nginx:1.26-alpine
```

Die Struktur (netzwerk, volumes, healthcheck) bleibt gleich.

## Debugging

### Website antwortet nicht

```bash
# 1. Prüfe Container-Status
docker compose ps

# 2. Prüfe Logs
docker compose logs website

# 3. Exec in Container
docker exec -it lld-website sh

# 4. Test innerhalb Container
wget http://localhost/
```

### Nginx-Fehler in Logs

```bash
docker compose logs website | grep -i error
```

### Keine Verbindung vom Proxy

```bash
# Test Netzwerk
docker exec -it lld-nginx ping website

# Test DNS-Auflösung
docker exec -it lld-nginx nslookup website
```

## Performance

- **Alpine Image:** ~12 MB (vs. ~200 MB Debian)
- **Startup Zeit:** <1 Sekunde
- **Memory:** ~10-20 MB
- **Gzip:** Aktiv für Text-Assets (automatisch)

## Sicherheit

- ✅ Keine öffentlichen Ports (nur über Proxy)
- ✅ Read-Only Volumes (keine Write-Zugriffe)
- ✅ Security Headers (XSS, Clickjacking-Protection)
- ✅ No Root User (Nginx Standard User)
- ✅ Alpine (minimale Oberfläche für Exploits)

## Lizenzen

- Nginx: 2-Clause BSD License
- Alpine: GPL v2 + Apache 2.0 (Packages)

