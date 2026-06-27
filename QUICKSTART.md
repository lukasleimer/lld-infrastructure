# Quick Start — LLD Infrastruktur

**Schnelleinstieg in weniger als 5 Minuten**

## 1. Vorbereitung (einmalig)

```bash
# Repository klonen (falls noch nicht vorhanden)
cd ~/Documents/lld/Repository/lld-infrastructure

# Environment-Datei erstellen
cp .env.example .env

# .env mit Passwort anpassen (wichtig!)
# Öffne .env und ändere:
#   POSTGRES_PASSWORD=dein_sicheres_passwort_16_zeichen_mindestens
```

**Anforderungen:**
- Docker & Docker Compose installiert
- Port 80 verfügbar
- ~1 GB freier Speicher

## 2. Infrastruktur starten

```bash
# Alle Services starten
docker compose up -d

# Status prüfen
docker compose ps

# Sollte zeigen:
# lld-postgres  ...  Up (healthy)
# lld-nginx     ...  Up (healthy)  
# lld-website   ...  Up (healthy)
```

## 3. Testen

```bash
# Website im Browser öffnen
http://localhost

# Oder via curl
curl http://localhost/

# Health-Check
curl http://localhost/health
```

**Erwartet:**
- Website mit "LLD Infrastructure" und grünem "✓ Running"
- Health-Endpoint antwortet mit "200 OK"

## 4. Services verwalten

```bash
# Logs anschauen
docker compose logs -f

# Nur Nginx-Logs
docker compose logs -f nginx

# Services stoppen (Daten bleiben!)
docker compose stop

# Services starten
docker compose start

# Alles stoppen & löschen (aber nicht Daten!)
docker compose down

# Alles löschen inkl. Daten (⚠️ VORSICHT!)
docker compose down -v
```

## 5. Troubleshooting

```bash
# Compose-Datei validieren
docker compose config

# Detaillierte Service-Informationen
docker ps -a
docker logs lld-postgres
docker logs lld-nginx
docker logs lld-website

# Netzwerk-Debug
docker exec lld-nginx ping website
docker exec lld-nginx curl http://website/

# Volume-Info
docker volume ls
```

## 6. Was läuft?

| Container | Port | Erreichbar | Zweck |
|-----------|------|-----------|-------|
| lld-postgres | 5432 | Nur intern | Datenbank |
| lld-nginx | 80 | http://localhost | Reverse Proxy |
| lld-website | - | Nur intern | Test-Website |

## 7. Wichtige Dateien

- `docker-compose.yml` — Root (Einstiegspunkt)
- `.env` — Secrets (nicht in Git!)
- `.env.example` — Template (in Git)
- `docs/sprints/` — Dokumentation
- `DOCKER_COMPOSE_README.md` — Detailliertes Manual

## 8. Häufige Fehler

| Fehler | Lösung |
|--------|--------|
| "Port 80 already in use" | Anderen Service auf Port 80 stoppen oder Port in docker-compose.yml ändern |
| "docker: command not found" | Docker installieren |
| "permission denied" | Mit sudo ausführen oder docker-Gruppe beitreten |
| Postgres nicht startet | .env anpassen, Container-Logs prüfen |
| Website zeigt 503 | Nginx-Logs prüfen, Website-Container muss healthy sein |

## 9. Nächste Schritte

Nach erfolgreichem Start:
1. Logs überprüfen: `docker compose logs`
2. Website testen: `curl http://localhost/`
3. Dokumentation lesen: `docs/sprints/sprint-006-orchestration.md`
4. Panel hinzufügen: `docs/sprints/sprint-007-panel.md` (später)

## Hilfe

- **Detaillierte Docs:** [DOCKER_COMPOSE_README.md](DOCKER_COMPOSE_README.md)
- **Architektur:** [docs/architecture/](docs/architecture/)
- **Standards:** [docs/standards/](docs/standards/)
- **Sprints:** [docs/sprints/](docs/sprints/)

---

**Status:** ✅ Ready to Go!  
**Getestet:** 2026-06-27
