# Netzwerkarchitektur — lld-infrastructure

Konzeptionelle Dokumentation der Netzwerkstruktur für die LLD-Infrastruktur.

## Ziel der Netzwerkarchitektur

Die Netzwerkarchitektur verfolgt folgende Ziele:

### Sicherheit
Das Netzwerk ist strukturiert, um Angriffsvektoren zu minimieren. Nicht notwendige Verbindungen sind nicht erlaubt.

### Isolation
Komponenten sind voneinander isoliert. Ein Sicherheitsleck in einer Komponente beeinträchtigt nicht automatisch andere.

### Wartbarkeit
Das Netzwerk ist nachvollziehbar und dokumentiert. Neue Komponenten können hinzugefügt werden, ohne bestehende zu beeinflussen.

### Erweiterbarkeit
Neue Dienste (z.B. Redis, Elasticsearch) können hinzugefügt werden, ohne das bestehende Netzwerk zu verändern.

### Klare Kommunikation
Die Kommunikationswege zwischen Diensten sind explizit definiert und dokumentiert.

## Netzwerkprinzipien

### Das Docker-Netzwerk `lld`

Alle Plattformdienste befinden sich im selben Docker-Netzwerk `lld`. Dies ermöglicht interne Kommunikation zwischen Containern.

Das Netzwerk ist **nicht** öffentlich erreichbar.

### Nur der Reverse Proxy ist öffentlich

Der Reverse Proxy ist die einzige Komponente, die von außen erreichbar ist.

Alle anderen Dienste sind ausschließlich intern erreichbar — direkte Verbindungen von außen sind nicht möglich.

### Datenbanken sind intern

PostgreSQL und zukünftige Datenspeicher sind ausschließlich intern erreichbar.

Sie haben **keine** öffentliche Schnittstelle.

Nur Anwendungen im selben Netzwerk können sich verbinden.

### Container kommunizieren über Servicenamen

Container sprechen sich nicht über IP-Adressen an, sondern über Servicenamen (z.B. `postgres`, `redis`).

Dies ermöglicht:
- Austausch von Komponenten ohne Code-Änderungen
- DNS-basierte Service-Discovery
- Resilienz gegen Neustart

### Direkte Kommunikation nur zwischen benötigten Komponenten

Es gibt keine "Alles mit Allem"-Konnektivität.

Jede Kommunikation wird explizit definiert. Dies minimiert Angriffsvektoren und Abhängigkeiten.

## Netzwerkübersicht

Die logische Struktur der Netzwerk-Schichten:

```
Internet
    ↓
Router
    ↓
Reverse Proxy
    │
    ├→ HTTP/HTTPS entschlüsselt
    ├→ Requests weitergeleitet
    ├→ Antworten verschlüsselt
    ↓
Docker Network (lld)
    │
    ├── panel.lukasleimer.dev (Django)
    ├── lukasleimer.dev (Django)
    ├── Kundenwebseiten (Container)
    ├── PostgreSQL (Datenbank)
    └── [Zukünftige Services]
    ↓
Persistente Volumes
    ├── Datenbankdaten
    ├── Uploads
    └── Caches
```

**Wichtig:** Das Docker-Netzwerk ist eine **Grenze**. Verbindungen über diese Grenze hinweg (in/out) erfolgen ausschließlich über den Reverse Proxy.

## Kommunikationsregeln

### Reverse Proxy ↔ Anwendungen

Der Reverse Proxy leitet eingehende HTTP(S)-Requests an die Anwendungen weiter.

**Ziel:** HTTP/HTTPS-Routing und Load Balancing

**Richtung:** Bidirektional (Request/Response)

**Sichtbarkeit:** Der Reverse Proxy muss alle Anwendungen erreichen können.

### Anwendungen ↔ PostgreSQL

Anwendungen verbinden sich mit der Datenbank, um persistente Daten zu speichern und zu lesen.

**Ziel:** Datenpersistenz

**Richtung:** Bidirektional (Query/Response)

**Sichtbarkeit:** Nur Anwendungen können PostgreSQL erreichen. Nicht der Reverse Proxy oder von außen.

### Anwendungen ↔ Zukünftige Services (z.B. Redis)

Künftige Dienste wie Redis (Caching) oder Elasticsearch (Suche) werden hinzugefügt.

**Ziel:** Optimierung (Caching, Suche, etc.)

**Richtung:** Bidirektional (Command/Response)

**Sichtbarkeit:** Nur relevante Anwendungen können diese Services erreichen.

### Monitoring ↔ Plattformdienste

Monitoring-Systeme (Prometheus, ELK) müssen die Metriken und Logs der Services sammeln.

**Ziel:** Observability

**Richtung:** Bidirektional (Metrics Pull / Logs Push)

**Sichtbarkeit:** Monitoring-Systeme sind intern, können alle Services erreichen.

### Bewusst nicht erlaubte Verbindungen

**Nicht möglich:**

- Externe Systeme → PostgreSQL (Datenbank ist nicht öffentlich)
- Externe Systeme → Redis/Cache (Caches sind nicht öffentlich)
- Anwendung A → PostgreSQL einer anderen Anwendung (Datenisolation)
- Reverse Proxy → PostgreSQL (unnötig und unsicher)

Diese Einschränkungen sind **absichtlich** und schützen die Plattform.

## Sicherheitsprinzipien

### Prinzip der geringsten Rechte

Jede Komponente hat **nur** die Zugriffe, die sie benötigt.

Eine Anwendung kann die Datenbank der anderen Anwendung nicht erreichen.

Der Reverse Proxy kann nicht direkt auf die Datenbank zugreifen.

### Minimaler Angriffsvektor

Je weniger externe Schnittstellen, desto kleiner die Angriffsfläche.

Nur der Reverse Proxy ist extern erreichbar. Das minimiert das Risiko.

### Interne Kommunikation bleibt intern

Kommunikation zwischen Komponenten im Docker-Netzwerk läuft **nicht** durch den Reverse Proxy.

Interne Services können direkt miteinander sprechen (über den internen Docker DNS).

Dies reduziert Latenz und Komplexität.

### Nur notwendige öffentliche Schnittstellen

Es gibt bewusst nur eine öffentliche Schnittstelle: der Reverse Proxy.

Alles andere erfolgt intern. Dies macht die Plattform sicher und wartbar.

## Skalierung und Zukünftige Entwicklung

Die Netzwerkarchitektur ist so gestaltet, dass sie mit der Plattform wachsen kann:

### Neue Anwendungen hinzufügen

Neue Kundenwebseiten oder Services können hinzugefügt werden, ohne die bestehende Netzwerk-Struktur zu ändern.

Sie verbinden sich einfach mit dem `lld`-Netzwerk und sind automatisch integriert.

### Neue Datenspeicher

Künftig können Redis, Elasticsearch oder andere Services hinzugefügt werden.

Sie werden wie PostgreSQL behandelt: intern, mit definierten Zugriffsregeln.

### Mehrere Anwendungs-Instanzen

Wenn eine Anwendung skaliert werden muss, können mehrere Container-Instanzen laufen.

Der Reverse Proxy distribuiert Requests automatisch (Load Balancing).

### Monitoring-Systeme ausbauen

Von Basis-Monitoring zu erweiterten Systemen (Alerting, Tracing, etc.).

Das Netzwerk bleibt unveändert — nur zusätzliche Service-Verbindungen.

### Mehrere Server (Zukunft)

Die grundlegenden Netzwerk-Prinzipien bleiben auch bei Multi-Server-Setup bestehen:
- Separation of Concerns
- Minimal öffentliche Schnittstellen
- Explizite Kommunikationswege

## Zusammenfassung

Die Netzwerkarchitektur der LLD-Infrastruktur folgt bewährten Sicherheits- und Wartbarkeitsprinzipien:

- **Einfach** — Klare Struktur mit definierten Kommunikationswegen
- **Sicher** — Minimale Angriffsfläche, Isolation zwischen Komponenten
- **Wartbar** — Nachvollziehbare und dokumentierte Kommunikation
- **Erweiterbar** — Neue Services können ohne Umbauten hinzugefügt werden

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/deployment.md](deployment.md) — Deployment-Architektur
