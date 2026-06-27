# Docker Compose — Zentrale Orchestrierung
#
# Diese Datei orchestriert alle Services der LLD-Infrastruktur.
#
# Sie ist der zentrale Einstiegspunkt für Entwicklung, Testing und Produktion.
#
# Struktur:
# - Zentrale Netzwerk- und Volume-Definition
# - Modulare Service-Includes
# - Konsistente Konfiguration über alle Services
#
# Die einzelnen Service-Dateien:
# - docker/compose/database.yml       — PostgreSQL Datenbank
# - docker/compose/proxy.yml          — Nginx Reverse Proxy
# - docker/compose/website.yml        — Test-Website
# - docker/compose/panel.yml          — Panel (später)
# - docker/compose/monitoring.yml     — Monitoring (später)

## Verwendung

### Alle Services starten
```bash
docker compose up -d
```

### Service-Status prüfen
```bash
docker compose ps
```

### Logs anschauen
```bash
# Alle Services
docker compose logs -f

# Nur Nginx
docker compose logs -f nginx

# Nur Datenbank
docker compose logs -f postgres

# Nur Website
docker compose logs -f website
```

### Services stoppen
```bash
docker compose down
```

### Services stoppen und Daten löschen
```bash
docker compose down -v
```

## Services (aktuell laufen)

### 1. PostgreSQL (lld-postgres)
- Image: `postgres:16.3-alpine`
- Netzwerk: `lld` (intern)
- Ports: Keine (nur intern erreichbar)
- Volume: `lld_postgres_data` (/var/lib/postgresql/data)
- Status: ✅ Läuft

### 2. Nginx Reverse Proxy (lld-nginx)
- Image: `nginx:1.26-alpine`
- Netzwerk: `lld` (intern)
- Ports: `80:80` (HTTP öffentlich)
- Volumes:
  - `docker/nginx` → `/etc/nginx/conf.d`
  - `lld_nginx_logs` → `/var/log/nginx`
- Status: ✅ Läuft

### 3. Website Test-Service (lld-website)
- Image: `nginx:1.26-alpine`
- Netzwerk: `lld` (intern)
- Ports: Keine (nur über Reverse Proxy)
- Volumes:
  - `docker/website/html` → `/usr/share/nginx/html` (statische HTML)
  - `docker/website/default.conf` → `/etc/nginx/conf.d/default.conf`
- Status: ✅ Läuft

## Netzwerk

Alle Services sind im Docker-Netzwerk `lld` verbunden:

```
┌─────────────────────────────────────────────┐
│         Docker-Netzwerk: lld                │
├─────────────────────────────────────────────┤
│                                             │
│  lld-postgres    lld-nginx    lld-website   │
│  (Port 5432)     (Port 80)    (Port 80 int) │
│   ↓              ↓             ↓            │
│   └──────────────┴─────────────┘            │
│                                             │
└─────────────────────────────────────────────┘
         ↓
    (Nur Nginx port 80 öffentlich)
```

Service-Kommunikation:
- PostgreSQL: `postgres` (von anderen Services)
- Nginx: `nginx` (falls nötig)
- Website: `website` (von Nginx Proxy)

## Volumes

Persistente Volumes (bleiben erhalten auch wenn Container stoppt):

- `lld_postgres_data` — PostgreSQL Datenbankdaten
- `lld_nginx_logs` — Nginx Access/Error Logs (später)

## Konfiguration

### Umgebungsvariablen

Konfiguration über `.env` Datei im Root:

```bash
# Template kopieren
cp .env.example .env

# .env editieren und mit Werten füllen
# (Besonders POSTGRES_PASSWORD!)
```

### Netzwerk-Konfiguration

Alle Services verwenden das externe Netzwerk `lld`:
- Bridge-Driver (default)
- Service-Discovery über DNS (Servicenamen)

### Volume-Konfiguration

Alle Volumes sind benannt (nie anonym):
- Persistent über Container-Restarts
- Manuell verwaltet (nicht automatisch gelöscht)

## First-Run / Setup

1. Repository klonen
2. `.env.example` zu `.env` kopieren
3. `.env` anpassen (besonders Passwörter!)
4. `docker compose up -d` ausführen
5. Healthchecks überprüfen: `docker compose ps`
6. Logs überprüfen: `docker compose logs`

## Testing

### Nginx Healthcheck
```bash
curl http://localhost/health
# Erwartet: 200 OK
```

### Website über Nginx
```bash
curl http://localhost/
# Erwartet: LLD Infrastructure Test-Seite
```

### Website direkt (aus anderem Container)
```bash
docker exec lld-nginx wget http://website/ -O -
# Erwartet: HTML-Seite
```

### Datenbank-Verbindung
```bash
docker exec lld-postgres psql -U postgres -d lld_panel -c "SELECT NOW();"
# Erwartet: Aktuelle Zeit
```

## Architektur Überblick

### Request-Flow (aktuell)

```
1. Client außen
    ↓
2. curl http://localhost/
    ↓
3. Nginx (lld-nginx) Port 80
    ↓
4. Proxy-Pass zu http://website/
    ↓
5. Website-Service (lld-website) internal
    ↓
6. Serviert HTML
    ↓
7. Response zurück zu Client
```

### Skalierung (später)

- Kundenwebseiten: `customer-{id}-*` Services
- Multi-Domain-Routing: Nginx Config
- Load Balancing: Zukünftig
- Clustering: Zukünftig

## Troubleshooting

### Container startet nicht

```bash
# Logs anschauen
docker compose logs <service>

# Container inspizieren
docker inspect lld-postgres

# Netzwerk-Probleme
docker network inspect lld
```

### Volume-Probleme

```bash
# Volumes auflisten
docker volume ls

# Volume inspizieren
docker volume inspect lld_postgres_data

# Volumes löschen (ACHTUNG: Daten weg!)
docker volume rm lld_postgres_data
```

### Netzwerk-Probleme

```bash
# Connectivität prüfen
docker exec lld-nginx ping website

# DNS-Auflösung
docker exec lld-nginx nslookup website

# Port-Bindungen
docker port lld-nginx
```

## Wichtige Dateien

- `docker-compose.yml` — Diese Datei (Einstiegspunkt)
- `docker/compose/*.yml` — Service-Definitionen
- `.env` — Umgebungsvariablen (nicht in Git!)
- `.env.example` — Template (in Git)
- `docker/nginx/` — Nginx-Konfiguration
- `docker/website/` — Website-Test-Dateien
- `docs/` — Dokumentation

## Nächste Schritte

- [ ] SSL/HTTPS (Sprint 5)
- [ ] Domain-Routing (Sprint 5)
- [ ] Panel-Service (Sprint 6+)
- [ ] Backup-Automatisierung (Sprint 7)
- [ ] Monitoring (Sprint 8)
- [ ] Customer Hosting (Sprint 9)

---

**Status:** ✅ Produktionsreife Basis  
**Datum:** 2026-06-27
