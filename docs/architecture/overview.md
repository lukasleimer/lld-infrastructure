# Architekturübersicht — lld-infrastructure

Zentrale Dokumentation der Infrastruktur-Architektur für die LLD-Plattform.

## Zweck dieses Repositories

`lld-infrastructure` verwaltet die **vollständige Infrastruktur** der LLD-Plattform. Es ist das zentrale Repository für alle infrastrukturellen Komponenten: von Containerisierung über Netzwerke bis zu Deployments und Backups.

Dieses Repository stellt die Infrastruktur bereit für:

- **panel.lukasleimer.dev** — Admin-Panel der Plattform
- **lukasleimer.dev** — Portfolio und Unternehmenswebseite
- **Kundenwebseiten** — Beliebig viele zukünftige Kundenprojekte

Ziel ist es, dass jede Infrastruktur-Komponente aus diesem Repository reproduzierbar ist.

## Architekturziele

Die Architektur dieses Projekts verfolgt folgende Ziele:

### Infrastructure as Code
Die gesamte Infrastruktur ist versioniert und als Code definiert. Jede Komponente ist reproduzierbar und nachvollziehbar.

### Reproduzierbarkeit
Der komplette Server kann zu jedem Zeitpunkt aus diesem Repository neu aufgebaut werden. Keine manuellen Konfigurationen, keine Seiteneffekte.

### Wartbarkeit
Infrastruktur muss verständlich und wartbar sein. Dokumentation ist gleichwertig mit Code. Technische Schulden werden vermieden.

### Modularität
Komponenten sind in sich geschlossen und unabhängig. Änderungen an einer Komponente beeinflussen nicht andere.

### Skalierbarkeit
Die Infrastruktur ermöglicht es, von einer bis zu beliebig vielen Anwendungen zu skalieren — ohne fundamentale Umbauten.

### Sicherheit
Sicherheit ist eingebaut, nicht nachgelagert. TLS, Isolation, Zugriffskontrolle sind Standards.

### Dokumentation als Architektur
Jede Entscheidung wird dokumentiert. ADRs, Designdokumente und Runbooks sind essenziell.

## Verantwortlichkeiten

### Dieses Repository verwaltet:

- **Docker** — Containerisierung aller Anwendungen
- **Docker Compose** — Orchestrierung der Container
- **Reverse Proxy** — HTTP(S)-Routing und Load Balancing
- **Netzwerke** — Isolation und Kommunikation zwischen Containern
- **Persistente Daten** — Volumes für Datenbankdaten, Uploads, Caches
- **SSL/TLS** — Verschlüsselung und Zertifikatverwaltung
- **Deployment** — Automatisierte Deployments und Rollbacks
- **Backups** — Datensicherung und Disaster Recovery
- **Monitoring** — Überwachung, Alerting und Logging

### Dieses Repository verwaltet NICHT:

- **Business-Logik** — Gehört in die Anwendungen (lld-panel, Kundenwebseiten)
- **Django-Code** — Applikationscode in separaten Repositories
- **Kundenverwaltung** — Teil des Admin-Panels
- **Website-Inhalte** — Templates, Assets und Inhalte in den Anwendungen
- **Datenbankmigrationen** — Gehören in die Anwendungen

Diese Trennung ermöglicht es, dass Infrastruktur und Anwendungen unabhängig entwickelt werden.

## Systemübersicht

Die LLD-Infrastruktur folgt einer klassischen, bewährten Architektur:

```
Internet
    ↓
Router
    ↓
Reverse Proxy (Nginx)
    ↓
Docker Network
    ↓
Container
    ├── panel.lukasleimer.dev (Django)
    ├── lukasleimer.dev (Django)
    ├── Kundenwebseiten (Docker)
    └── PostgreSQL (Datenbankserver)
    ↓
Persistente Volumes
    ├── Datenbankdaten
    ├── Uploads
    ├── Caches
    └── Konfigurationen
```

Diese einfache, nachvollziehbare Architektur ermöglicht:
- Hohe Zuverlässigkeit
- Einfache Verwaltung
- Vorhersagbare Performance
- Sicherheit durch Isolation

## Architekturprinzipien

### Architecture First
Architektonische Entscheidungen werden **vor** der Implementierung getroffen und dokumentiert. Dies vermeidet später teure Umbauten.

### Documentation First
Dokumentation ist nicht "schön zu haben", sondern Teil der Architektur. Jede Entscheidung wird dokumentiert (ADRs).

### Framework First
Frameworks und Standards werden bewusst gewählt. Dies führt zu Konsistenz und ermöglicht neuen Team-Mitgliedern schnellen Einstieg.

### Eine Verantwortung pro Modul
Jede Komponente hat eine klare, definierte Aufgabe. Keine Über-Engineering, keine versteckten Abhängigkeiten.

### Git als Single Source of Truth
Der Zustand des Systems ist ausschließlich in Git definiert. Keine manuellen Out-of-Band-Änderungen.

### Reproduzierbare Deployments
Jeder Deployment ist identisch reproduzierbar. Deteministisch, nicht zufällig.

### Keine technischen Schulden
Schnelle Fixes werden vermieden. Jede Lösung wird langfristig gedacht. Wartbarkeit vor Geschwindigkeit.

### Keine Workarounds
Workarounds führen zu Komplexität. Stattdessen wird die eigentliche Ursache gelöst.

## Projektbeziehungen

### lld-infrastructure ↔ lld-panel

**lld-infrastructure** stellt die **Plattform** bereit.

**lld-panel** ist eine **Anwendung**, die auf dieser Plattform läuft. Das Panel nutzt die Infrastruktur, verwandelt diese aber nicht.

Trennung der Concerns:
- **Infrastruktur** — dieses Repository
- **Applikation** — lld-panel Repository

### lld-infrastructure ↔ Kundenanwendungen

**lld-infrastructure** definiert die **Umgebung** für Kundenwebseiten.

**Kundenwebseiten** sind **Mieter** (Tenants) auf dieser Plattform.

Die Infrastruktur ermöglicht:
- Isolation zwischen Kundenwebseiten
- Ressourcen-Limits
- Automatische Backups
- SSL-Verwaltung pro Kunde

## Weitere Architekturdokumente

Diese Übersicht bietet die **hohe Ebene** der Architektur.

Details werden in separaten Dokumenten beschrieben:

- **[docs/architecture/](../architecture/)** — Spezifische Architekturdokumente
- **[docs/adr/](../adr/)** — Architecture Decision Records für jede Entscheidung
- **[docs/standards/](../standards/)** — Standards und Konventionen

Weitere Lektüre:
- [ROADMAP.md](../../ROADMAP.md) — Entwicklungsplanung
- [README.md](../../README.md) — Projekt-Übersicht

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27
