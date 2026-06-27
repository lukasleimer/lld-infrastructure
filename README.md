# lld-infrastructure

Vollständige Infrastruktur einer Ein-Mann-Webagentur — reproducible, dokumentiert und wartbar.

## Projektziel

`lld-infrastructure` definiert die gesamte Infrastruktur für den Betrieb von Webanwendungen. Das Repository folgt dem Prinzip **Infrastructure as Code** und ermöglicht es, den kompletten Server ausschließlich aus diesem Repository reproduzierbar aufzubauen.

Langfristig werden darüber folgende Anwendungen betrieben:

- **panel.lukasleimer.dev** — Admin-Panel
- **lukasleimer.dev** — Portfolio / Unternehmenswebseite
- **Kundenwebseiten** — beliebig viele Kundenprojekte

## Projektphilosophie

Die Entwicklung dieses Repositories folgt unverrückbaren Grundsätzen:

- **Architecture First** — Architektur vor Implementierung
- **Documentation First** — Dokumentation ist gleichwertig mit Code
- **Framework First** — Bewusste Auswahl von Frameworks und Standards
- **Keine technischen Schulden** — Langfristigkeit statt schnelle Fixes
- **Keine Workarounds** — Lösungen, nicht Umwege
- **Wartbarkeit vor Geschwindigkeit** — Verständlichkeit ist Priorität
- **Git ist die Quelle der Wahrheit** — Alle Zustände sind versioniert
- **Reproduzierbare Deployments** — Jeder Zustand ist rekonstruierbar

## Inhalt dieses Repositories

- Docker-Infrastruktur und Containerisierung
- Docker Compose Orchestrierung
- Reverse Proxy Konfiguration
- Netzwerk-Definitionen
- Persistente Volumes und Daten
- SSL/TLS-Zertifikatsverwaltung
- Deployment-Prozesse
- Backup-Strategien
- Monitoring und Logging
- Infrastructure-as-Code Scripts

## Nicht in diesem Repository

Folgende Inhalte gehören **nicht** hier hin:

- Django-Applikationscode
- Kundenverwaltungssysteme
- Business-Logik
- Website-Templates und Frontend-Assets
- Datenbankmigrationen (außer Struktur)

## Zielarchitektur

```
Internet
    ↓
Router
    ↓
Reverse Proxy (Nginx)
    ↓
Docker Network
    ↓
Container (Applikationen)
    ↓
Persistente Volumes (Daten)
```

## Dokumentationsstruktur

### `docs/`
Zentrale Dokumentation aller Infrastruktur-Aspekte. Enthält Übersichten, Entscheidungen und Standards.

### `docs/architecture/`
Architektur-Übersichten und System-Designs. Beschreiben das Gesamtsystem und seine Komponenten.

### `docs/adr/`
Architecture Decision Records (ADR). Dokumentieren technische Entscheidungen mit Kontext, Alternativen und Begründung.

### `docs/decisions/`
Entscheidungsprotokolle und Begründungen für spezifische Choices im Projekt.

### `docs/standards/`
Festgelegte Standards und Konventionen für Konfiguration, Naming, Deployment und Dokumentation.

### `docs/sprints/`
Sprint-Planung und Implementierungs-Roadmap. Dokumentiert iterative Entwicklungsschritte.

### `docker/`
Sämtliche Infrastruktur-Definitionen. Organisiert nach Concerns:

- **compose/** — Docker Compose Konfigurationen
- **nginx/** — Reverse Proxy Konfigurationen (zukünftig)
- **monitoring/** — Monitoring und Observability (zukünftig)
- **ssl/** — SSL/TLS-Zertifikate und deren Verwaltung
- **backup/** — Backup-Strategien und Scripts
- **scripts/** — Hilfsscripte für Deployment und Verwaltung
- **volumes/** — Persistente Daten-Definitionen

## Roadmap

Die langfristige Planung und Meilensteine sind in [ROADMAP.md](ROADMAP.md) dokumentiert.

## Architektur und Entscheidungen

Sämtliche Architekturentscheidungen werden dokumentiert in:

- [docs/architecture/](docs/architecture/) — Systemübersichten und Designs
- [docs/adr/](docs/adr/) — Architecture Decision Records

Diese Dokumentationen bilden die Grundlage für alle technischen Entscheidungen.

## Status

Dieses Repository ist in **aktiver Entwicklung**. Die grundlegende Struktur ist festgelegt, die Implementierung erfolgt in iterativen Sprints.

Siehe [docs/sprints/](docs/sprints/) für den aktuellen Entwicklungsstand.
