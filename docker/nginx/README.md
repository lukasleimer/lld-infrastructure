# Nginx Configuration Structure — lld-infrastructure
#
# Das docker/nginx-Verzeichnis enthält die Nginx-Konfigurationsdateien.
#
# Diese werden von proxy.yml eingebunden:
#   volumes:
#     - ./../../docker/nginx:/etc/nginx/conf.d
#
# Dateien in diesem Verzeichnis:
#
# - default.conf      — Platzhalter und Health-Check Endpoint
# - [domains].conf    — Pro Domain ein Config-File (später)
# - ssl.conf          — SSL-Konfiguration (Sprint 5)
#
# Struktur (Zukunft):
#
# default.conf        — Health-Check und Fallback
# panel.conf          — Virtual Host für panel.lukasleimer.dev
# website.conf        — Virtual Host für lukasleimer.dev
# customers.conf      — Virtual Hosts für Kundendomains (dynamisch)
# ssl.conf            — Zentrale SSL-Konfiguration
#
# Wichtig:
# - Jede Datei mit .conf-Endung wird von Nginx geladen
# - Reload ohne Container-Neustart möglich (docker exec + nginx reload)
# - Konfiguration wird versioniert (Git)
# - Domains und Zertifikate sind dynamisch (später im Sprint 5+)
