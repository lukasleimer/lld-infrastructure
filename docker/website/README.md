# Website Service — lld-infrastructure
#
# Platzhalter-Website für Infrastruktur-Testing.
#
# Dateien:
#
# - index.html         — Test-Webseite (statisches HTML)
# - default.conf       — Nginx-Konfiguration
#
# Zweck:
# - Testing der Docker Compose Struktur
# - Verification des Nginx-Routings
# - End-to-End Request-Flow Test
# - Basis für zukünftige Website-Integration
#
# Später wird dieser Service durch die echte lld-website ersetzt:
# - Echtes Docker-Image mit Astro/Next.js/etc.
# - Echte Website-Inhalte
# - Produktionsnah
#
# Testing:
# curl http://localhost/       (direkt, wenn Port veröffentlicht)
# curl http://lld-website/     (aus anderem Container im Netzwerk)
