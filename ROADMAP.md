# Roadmap — lld-infrastructure

Langfristige Entwicklungsplanung für eine produktionsreife Infrastrukturplattform.

## Projektvision

**lld-infrastructure** transformiert die Infrastruktur einer Ein-Mann-Webagentur von fragmentierten Konfigurationen zu einem vollständig versionierten, dokumentierten und reproduzierbaren System.

Die Vision ist es, dass jede Infrastruktur-Komponente:
- In Git versioniert ist
- Durch Code dokumentiert wird
- Konsistent deploybar ist
- Wartbar und skalierbar bleibt

## Langfristige Ziele

- **100% Reproduzierbarkeit** — Der komplette Server kann aus diesem Repository neu aufgebaut werden
- **Operationalisierung** — Alle Betriebsprozesse sind automatisiert
- **Wartbarkeit** — Infrastruktur ist wartbar und erweiterbar
- **Skalierbarkeit** — Von einer Website zu beliebig vielen Kundenwebsiten
- **Sicherheit** — Moderne TLS, regelmäßige Backups, Monitoring
- **Transparenz** — Alle Entscheidungen sind dokumentiert

## Nicht-Ziele

- ❌ Kubernetes oder Orchestrierungsplattformen
- ❌ Multi-Server-Setup in dieser Phase
- ❌ Automated Scaling
- ❌ Komplexe Microservices-Architektur
- ❌ Infrastructure-Abstraktionen (Terraform o.ä.) in Phase 1

Fokus: **Single-Server-Excellence**

---

## Meilensteine

### Phase 1: Foundation

**Ziel:** Grundstruktur und Repository-Standards etablieren

- ✓ Repository-Struktur definieren
- ✓ Dokumentationsframework aufbauen
- ✓ Standards und Konventionen festlegen
- ✓ Git-Workflow etablieren
- Architecture Decision Records (ADRs) initialisieren

**Ergebnis:** Saubere Basis für alle weiteren Phasen

---

### Phase 2: Docker Compose

**Ziel:** Containerisierung und lokale Orchestrierung aufbauen

- Docker Compose Konfigurationen implementieren
- Netzwerk-Topologie definieren
- Container-Standards festlegen
- Health Checks konfigurieren
- Development-Setup dokumentieren

**Ergebnis:** Lokales Development-Environment vollständig in Docker

---

### Phase 3: Reverse Proxy

**Ziel:** HTTP(S)-Traffic-Management und Routing

- Nginx als Reverse Proxy konfigurieren
- Virtual Hosts für alle Anwendungen
- HTTP → HTTPS Redirection
- Basic Load Balancing
- Rate Limiting und DDoS-Protection (grundlegend)

**Ergebnis:** Professionales HTTP(S)-Routing aller Services

---

### Phase 4: PostgreSQL

**Ziel:** Datenbankinfrastruktur und Persistenz

- PostgreSQL Container-Setup
- Persistente Volume-Verwaltung
- Datenbankinitialisierungsskripte
- Benutzer- und Privileges-Management
- Backup-Vorbereitung

**Ergebnis:** Produkive Datenbankinfrastruktur mit Persistenz

---

### Phase 5: SSL/TLS

**Ziel:** Verschlüsselte Kommunikation und Zertifikatverwaltung

- Let's Encrypt Integration
- Zertifikat-Automation (Certbot)
- Auto-Renewal-Prozesse
- Zertifikats-Monitoring
- HSTS und Security Headers

**Ergebnis:** Moderne, automatisierte TLS-Infrastruktur

---

### Phase 6: Deployment

**Ziel:** Automatisierte und reproduzierbare Deployments

- Deployment-Scripts
- Version-Management
- Rolling Updates
- Rollback-Strategien
- GitHub Actions Integration (zukünftig)

**Ergebnis:** Sichere und nachvollziehbare Deployments

---

### Phase 7: Backup & Recovery

**Ziel:** Datensicherung und Disaster-Recovery

- Backup-Strategie definieren (3-2-1-Regel)
- Automatisierte Datenbank-Backups
- Volume-Backups
- Off-Site-Backup-Speicherung (S3 o.ä.)
- Recovery-Tests und Dokumentation

**Ergebnis:** Produktive Backup-Infrastruktur mit regelmäßigen Tests

---

### Phase 8: Monitoring & Logging

**Ziel:** Überwachung und Observability

- Prometheus für Metriken
- ELK-Stack oder Loki für Logging
- Alerting-Regeln
- Dashboard-Setup
- Incident-Response-Dokumentation

**Ergebnis:** Vollständige Observability über alle Services

---

### Phase 9: Customer Hosting

**Ziel:** Multi-Tenant-Fähigkeit für Kundenwebsiten

- Kundenwebsiten in Containern isolieren
- Datenbank-Isolation
- Ressourcen-Limits
- DNS-Verwaltung
- Kunden-spezifische SSL-Zertifikate

**Ergebnis:** Sichere und skalierbare Kundenwebseiten-Plattform

---

### Phase 10: LLD Panel Integration

**Ziel:** Admin-Panel als zentrale Verwaltungschnittstelle

- Panel-Container in Infrastruktur integrieren
- API-Endpoints für Infrastruktur-Verwaltung
- Dashboard für Monitoring und Status
- Kunden-Management-Interface
- Audit-Logging

**Ergebnis:** Zentrale Admin-Schnittstelle für alle Infrastruktur

---

### Phase 11: Production Release

**Ziel:** Infrastruktur ist produktionsreif und wartbar

- Vollständige Dokumentation
- Runbooks für alle Operationen
- Performance-Optimierung
- Security Audit
- SLA-Definition
- Prozesse etabliert

**Ergebnis:** Produktionsreife Infrastrukturplattform

---

## Timeline

| Phase | Ziel | Status |
|-------|------|--------|
| 1 | Foundation | 🟢 In Progress |
| 2 | Docker Compose | 🔴 Geplant |
| 3 | Reverse Proxy | 🔴 Geplant |
| 4 | PostgreSQL | 🔴 Geplant |
| 5 | SSL/TLS | 🔴 Geplant |
| 6 | Deployment | 🔴 Geplant |
| 7 | Backup & Recovery | 🔴 Geplant |
| 8 | Monitoring & Logging | 🔴 Geplant |
| 9 | Customer Hosting | 🔴 Geplant |
| 10 | LLD Panel Integration | 🔴 Geplant |
| 11 | Production Release | 🔴 Geplant |

## Wichtige Hinweise

### Roadmap-Flexibilität

Diese Roadmap dient als Orientierungshilfe und wird im Laufe der Entwicklung angepasst:

- Phasen können sich überlappen
- Prioritäten können sich verschieben
- Neue Erkenntnisse führen zu Anpassungen
- Jede Phase führt zu neuen Erkenntnissen über die nächsten Phasen

**Werkzeug für die Planung, nicht Gesetz.**

### Architecture First

Jede Phase wird durch ein Architecture Decision Record (ADR) eingeleitet, das die technischen Entscheidungen dokumentiert. Siehe [docs/adr/](docs/adr/).

### Dokumentation

Zu jeder Phase gehört:
- Dokumentation in [docs/architecture/](docs/architecture/)
- ADRs für technische Decisions
- Sprint-Planung in [docs/sprints/](docs/sprints/)
- Implementierungs-Details in Code-Comments

## Feedback & Changes

Fragen, Ideen und Änderungsvorschläge sind willkommen. Neue Erkenntnisse führen zu neuen ADRs und Roadmap-Updates.

---

**Letztes Update:** 2026-06-27
