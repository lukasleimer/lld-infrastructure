# Reverse-Proxy-Architektur — lld-infrastructure

Konzeptionelle Dokumentation der Reverse-Proxy-Komponente als zentrale Eingangskomponente der LLD-Infrastruktur.

## Zweck des Reverse Proxy

Der Reverse Proxy ist der **einzige öffentliche Einstiegspunkt** der gesamten Infrastruktur.

Jede HTTP/HTTPS-Anfrage von außen (Internet) durchläuft den Reverse Proxy. Von dort wird sie an die relevante interne Anwendung weitergeleitet.

Dies hat mehrere Konsequenzen:

- **Anwendungen sind nicht direkt öffentlich erreichbar** — Sie befinden sich hinter dem Proxy im internen Netzwerk
- **Zentrale Kontrolle** — Alle öffentlichen HTTP(S)-Anfragen gehen durch eine einzige Komponente
- **Sicherheit** — Der Proxy schützt die dahinter liegenden Dienste

## Ziele

### Zentraler Einstiegspunkt

Ein Single Point of Entry für die gesamte Plattform. Dies vereinfacht Verwaltung, Sicherheit und Monitoring.

### Domainbasiertes Routing

Verschiedene Domains werden auf verschiedene interne Dienste geroutet:

- `panel.lukasleimer.dev` → Admin-Panel
- `lukasleimer.dev` → Portfolio-Website
- `kunde.at`, `kunde.de`, etc. → Kundenwebseiten

Der Proxy entscheidet basierend auf der eingehenden Domain, wohin der Request gehört.

### TLS-/SSL-Terminierung

Verschlüsselte Kommunikation (HTTPS) wird vom Proxy gemanagt.

- Inbound: HTTPS vom Internet wird entschlüsselt
- Inbound-Processing: Der Proxy verarbeitet unverschlüsselt
- Outbound: Kommunikation zu internen Services (unverschlüsselt, da im Netzwerk)

Dies ermöglicht, dass interne Services nicht selbst HTTPS verwalten müssen.

### Entkopplung der Anwendungen

Anwendungen müssen nicht wissen, dass sie hinter einem Proxy laufen.

Sie antwortet auf Requests — ob diese vom Proxy oder direkt kommen, ist ihnen egal.

Dies ermöglicht Flexibilität: Anwendungen können ausgetauscht werden, ohne andere zu beeinflussen.

### Einheitliche öffentliche Schnittstelle

Eine einzige Schnittstelle für alle externen Anfragen führt zu:
- Konsistenten Sicherheitsrichtlinien
- Einheitlichem Fehlerbehandlung
- Zentralem Logging und Monitoring

## Verantwortlichkeiten

### Entgegennahme eingehender HTTP/HTTPS-Anfragen

Der Reverse Proxy listenetn auf Standard-Ports und akzeptiert eingehende Anfragen von überall.

### Domainbasiertes Routing

Basierend auf der `Host`-Header der Anfrage wird entschieden, an welchen internen Service diese weitergeleitet wird.

### SSL-/TLS-Zertifikatverwaltung

Der Proxy verwaltet SSL-Zertifikate für alle öffentlichen Domains.

Dies umfasst:
- Zertifikat-Speicherung
- Gültigkeit und Ablauf
- Automatische Erneuerung

### Fehlerbehandlung

Wenn ein interner Service nicht erreichbar ist oder fehlt, zeigt der Proxy einen fehlergerechten HTTP-Status (z.B. 503 Service Unavailable).

### Logging (konzeptionell)

Der Proxy protokolliert eingehende Requests, Routing-Entscheidungen und Fehler.

Dies dient der späteren Analyse und Debugging.

## Nicht-Verantwortlichkeiten

### Business-Logik

Der Proxy führt **keine** Business-Logik aus.

Er routet Requests weiter — die Anwendungen dahinter verarbeiten diese.

### Deployment

Der Proxy startet und verwaltet nicht die dahinter liegenden Anwendungen.

Dies ist Aufgabe der Infrastruktur-Orchestrierung (Docker Compose).

### Datenbankzugriffe

Der Proxy greift nicht auf die Datenbank zu.

Nur die Anwendungen dahinter interagieren mit der Datenbank.

### Persistente Daten

Der Proxy speichert keine Anwendungsdaten.

Er ist ein durchleitendes System, nicht speichernd.

### Interne Anwendungslogik

Der Proxy enthält keine Logic zur Authentifizierung, Autorisierung oder Geschäftslogik.

Dies wird in den Anwendungen implementiert.

## Architekturübersicht

Die logische Struktur des Reverse Proxy und der dahinter liegenden Services:

```
Internet
    ↓
HTTPS Request (Host: panel.lukasleimer.dev)
    ↓
Reverse Proxy
    ├→ SSL entschlüsseln
    ├→ Host-Header prüfen
    ├→ Routing-Entscheidung treffen
    ↓
Routing
    │
    ├─ panel.lukasleimer.dev
    │       ↓
    │   lld-panel (Internal Docker Service)
    │
    ├─ lukasleimer.dev
    │       ↓
    │   lld-website (Internal Docker Service)
    │
    ├─ kunde.at
    │       ↓
    │   customer-001-website (Internal Docker Service)
    │
    └─ kunde.de
            ↓
        customer-002-website (Internal Docker Service)

    ↓
Internal Docker Network (lld)
    ↓
Application Processing
    ↓
Response zurück durch Proxy
    ↓
HTTPS-Response (encrypted) zurück an Client
```

Nur der Reverse Proxy-Container ist von außen erreichbar. Alle anderen Container sind im internen Netzwerk.

## Architekturprinzipien

### Nur ein öffentlicher Einstiegspunkt

Dies ist eine bewusste Architektur-Entscheidung.

**Vorteile:**
- Sicherheit: Alle öffentlichen Anfragen gehen durch einen Punkt
- Wartbarkeit: Nur ein Ort zum Konfigurieren von öffentlich
- Monitoring: Vollständiger Überblick über ein Punkt

**Alternative:** Jede Anwendung selbst öffentlich — würde Komplexität massiv erhöhen

### Anwendungen sind nicht direkt öffentlich erreichbar

Dies ist ein Sicherheits-Feature.

Die internen Services sind im `lld`-Netzwerk. Sie sind:
- Nicht von außen erreichbar
- Nicht direkt im Internet sichtbar
- Geschützt durch den Proxy

### Domainbasiertes Routing

Der Proxy routet basierend auf dem `Host`-Header (also der Domain in der URL).

Dies ermöglicht:
- Mehrere Anwendungen hinter einem Proxy
- Einfache Verwaltung von neuen Domains/Anwendungen
- Skalierbarkeit

### Einheitliche Sicherheitsrichtlinien

Ein Proxy für alles bedeutet: Sicherheitsrichtlinien werden einmal definiert und gelten für alles.

Dies umfasst:
- Rate Limiting
- DDoS-Protection
- Security Headers
- Monitoring

### Austauschbarkeit der Reverse-Proxy-Technologie

Die Architektur ist **nicht** an eine spezifische Technologie gebunden.

Andere Technologien könnte verwendet werden (z.B. Traefik, HAProxy) — die Architektur bliebe gleich.

Dies ermöglicht Flexibilität und verhindert Vendor Lock-In.

## Langfristige Entwicklung

Die Reverse-Proxy-Infrastruktur wird schrittweise erweitert:

### Phase 1: Statische Domain-Konfiguration

Domains und Routing werden manuell konfiguriert.

Jede neue Kundenwebseite erfordert manuelle Proxy-Konfiguration.

**Status:** Foundation-Phase

### Phase 2: Automatische Domain-Verwaltung

Das LLD Panel kann neue Domains registrieren und konfigurieren.

Der Proxy erhält neue Routing-Regeln ohne manuelle Eingriffe.

**Status:** Geplant nach Deployment-Phase

### Phase 3: Automatische SSL-Zertifikate

Zertifikate werden automatisch für neue Domains generiert.

Keine manuellen Zertifikat-Verwaltungen mehr.

**Status:** Geplant nach SSL-Phase

### Phase 4: Integration mit LLD Panel

Das Admin-Panel hat eine UI zur Domain-Verwaltung.

Neue Kundenwebseiten können über das Panel deployed werden — Proxy-Konfiguration erfolgt automatisch.

**Status:** Geplant nach LLD Panel Integration-Phase

### Phase 5: Erweiterbarkeit

Neue interne Services (z.B. API-Gateway, weitere Backoffices) können hinzugefügt werden.

Der Proxy routet diese automatisch basierend auf neuen Domains.

**Status:** Kontinuierliche Verbesserung

## Performance und Skalierung

Der Reverse Proxy ist eine **Durchleitungs-Komponente**, nicht ein Speicher oder Rechner.

Dies bedeutet:
- Minimale Latenz (Daten werden nicht modifiziert)
- Hoher Durchsatz (viele Requests möglich)
- Einfache Skalierung (Load Balancing möglich)

In späteren Phasen könnte es mehrere Proxy-Instanzen geben, die Load Balance betreiben.

## Zusammenfassung

Der Reverse Proxy ist die **Haustür** der LLD-Infrastruktur:

- **Einfach** — Ein Einstiegspunkt, klare Routing-Logik
- **Sicher** — Schützt interne Services, zentrale Sicherheitsrichtlinien
- **Wartbar** — Leicht zu konfigurieren und zu erweitern
- **Skalierbar** — Kann wachsen, wenn die Plattform wächst

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/networking.md](networking.md) — Netzwerkarchitektur
- [docs/architecture/deployment.md](deployment.md) — Deployment-Architektur
