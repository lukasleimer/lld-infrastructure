# Storage-Architektur — lld-infrastructure

Konzeptionelle Dokumentation der Storage- und Datenpersistenz-Strategie der LLD-Infrastruktur.

## Ziel der Storage-Architektur

Die Storage-Architektur verfolgt folgende Ziele:

### Persistenz
Daten überleben Container-Neustarts, Server-Neustarts und Deployments.

Einmal gespeicherte Daten sind nicht verloren.

### Datenintegrität
Daten werden zuverlässig gespeichert und können konsistent gelesen werden.

Keine Datenverluste, keine Korruption.

### Wartbarkeit
Die Datenverwaltung ist nachvollziehbar und wartbar.

Neue Datentypen können leicht hinzugefügt werden.

### Trennung von Anwendung und Daten
Container sind zustandslos (stateless).

Daten existieren außerhalb der Container und können wiederverwendet werden.

Dies ermöglicht es, Container zu stoppen, zu ersetzen und neu zu starten — ohne Daten zu verlieren.

### Wiederherstellbarkeit
Daten können zu einem früheren Zustand wiederhergestellt werden.

Dies ist essentiell für Disaster Recovery und Datenschutz.

## Datenkategorien

### Datenbanken

**Zweck:** Strukturierte, persistente Daten (Benutzer, Inhalte, Metadaten)

**Anforderungen:**
- Maximale Zuverlässigkeit
- ACID-Compliance
- Konsistenz
- Transaktionssicherheit

**Beispiel:** PostgreSQL

**Storage:** Dediziertes Database Volume

### Medien und Uploads

**Zweck:** Benutzerhochladene Dateien (Bilder, PDFs, Videos)

**Anforderungen:**
- Große Dateien verarbeiten
- Schneller Zugriff
- Versionierung (optional)
- Backup-fähig

**Beispiel:** Kundenbilder, Dokumente

**Storage:** Dediziertes Media/Upload Volume

### Konfiguration

**Zweck:** Umgebungsspezifische Konfigurationen (API-Keys, Datenbank-Credentials, Settings)

**Anforderungen:**
- Sicher (sensible Daten)
- Zentral verwaltbar
- Versionierbar

**Beispiel:** `.env`-Dateien, Config-Maps

**Storage:** Versionskontrolliert oder gesichert verwaltbar

### Logs

**Zweck:** Ereignisverlauf und Debugging-Informationen aller Services

**Anforderungen:**
- Zentrale Sammlung
- Durchsuchbar
- Archivierbar
- Kurzfristiges Retention

**Beispiel:** Application Logs, Reverse Proxy Logs, Database Logs

**Storage:** Dediziertes Log Volume oder Log-Aggregation-Service

### Backups

**Zweck:** Reproduktion des Systems zu früheren Zeitpunkten

**Anforderungen:**
- Regelmäßig erstellt
- Testbar (Recovery-Test)
- Redundant gelagert
- Langfristiges Retention

**Beispiel:** Tägliche Datenbank-Backups, Volume-Snapshots

**Storage:** Lokale Backups + Remote-Offsite-Storage

### Temporäre Daten

**Zweck:** Kurzfristige Zwischenspeicherung (Caches, Session-States, Temp-Dateien)

**Anforderungen:**
- Schneller Zugriff
- Nicht kritisch (Verlust ist akzeptabel)
- Automatisches Cleanup

**Beispiel:** In-Memory Caches, Session-Caches, Temp-Verzeichnisse

**Storage:** Flüchtige Volumes oder In-Memory (Redis)

## Speicherprinzipien

### Persistente Daten liegen außerhalb der Container

Container sind **Ephemeral** (flüchtig). Sie können jederzeit stoppen und gelöscht werden.

Persistente Daten liegen in Volumes, die **außerhalb** der Container existieren.

Dies ermöglicht es, Container zu ersetzen, ohne Daten zu verlieren.

### Container bleiben austauschbar

Weil Daten außerhalb liegen, sind Container austauschbar:
- Container A startet mit Volume X
- Container A wird gelöscht
- Container B startet mit dem gleichen Volume X
- Daten sind identisch

Dies ist essentiell für Skalierbarkeit und Wartbarkeit.

### Daten überleben Container-Neustarts

Container können neustarten (Deployment, Update, Fehler).

Daten in Volumes werden nicht gelöscht.

Das System kann vollständig neu starten — die Daten sind noch da.

### Es werden keine anonymen Volumes verwendet

Jedes Volume hat einen **eindeutigen Namen**, so dass es leicht identifizierbar ist.

Keine namenlosen, autonom verwalteten Volumes.

Dies verhindert "verlorene" Volumes und macht das System wartbar.

### Volumes folgen einer einheitlichen Namenskonvention

Volumes werden konsistent benannt, so dass es klar ist, wofür sie verwendet werden:
- `lld-postgres-data` — PostgreSQL Datenbankdaten
- `lld-uploads` — Benutzerdateien
- `lld-logs` — Anwendungs-Logs
- `customer-001-data` — Daten einer Kundenwebseite

Dies verbessert Wartbarkeit und vermeidet Verwechslungen.

## Architekturübersicht

Die logische Struktur von Anwendungen, Volumes und physischem Speicher:

```
Container Layer
    │
    ├── panel Container → lld-panel Process
    ├── website Container → lld-website Process
    ├── customer-001 Container → Application Process
    └── postgres Container → PostgreSQL Server
    │
    ↓ (Daten-Zugriff)
    │
Volume Layer
    │
    ├── lld-postgres-data Volume
    │       (PostgreSQL WAL, Heap-Dateien)
    │
    ├── lld-uploads Volume
    │       (Benutzerhochladene Medien)
    │
    ├── lld-logs Volume
    │       (Application-, System-, Database-Logs)
    │
    ├── customer-001-data Volume
    │       (Kundenspeichfische Daten)
    │
    └── lld-config Volume
            (Konfigurationsdateien)
    │
    ↓
    │
Host Filesystem
    │
    ├── /data/lld/postgres/
    ├── /data/lld/uploads/
    ├── /data/lld/logs/
    ├── /data/customer-001/
    └── /data/lld/config/
    │
    ↓
    │
Physischer Speicher
    │
    └── Filesystem (ext4, etc.)
```

**Wichtig:**
- Container sind oben (flüchtig)
- Volumes sind in der Mitte (unvergänglich)
- Host-Filesystem ist darunter (physischer Speicher)

Daten fließen von Containern in Volumes und werden auf dem Host-Filesystem persistiert.

## Verantwortlichkeiten

### Container

Container sind **zustandslos** und **flüchtig**:
- Führen Prozesse aus
- Lesen und schreiben zu Volumes
- Können jederzeit neu gestartet oder ersetzt werden
- Speichern **keine** persistenten Daten intern

### Docker Volumes

Volumes sind die **persistente Abstraktionsschicht**:
- Werden von Containern gemountet
- Existieren unabhängig von Containern
- Können zwischen Containern geteilt werden (wenn gewünscht)
- Werden vom Docker Daemon verwaltet

### Host-Dateisystem

Das Host-Dateisystem ist die **physische Persistenz**:
- Speichert tatsächlich die Bytes auf der Festplatte
- Wird von Volumes verwendet
- Kann mit Backups gesichert werden
- Kann auf verschiedenen Filesystemen liegen

### PostgreSQL

PostgreSQL ist der **Datenbankserver**:
- Verwaltet strukturierte Daten in seinem Volume
- Garantiert Konsistenz und ACID-Einhaltung
- Schreibt auf sein dediziertes Volume
- Hat keine Abhängigkeit von anderen Containern

### Anwendungen

Anwendungen sind die **Daten-Produzenten und Konsumenten**:
- Schreiben Daten in die Datenbank (via PostgreSQL)
- Laden Dateien in Media-Volumes
- Lesen Konfigurationen
- Sind ansonsten zustandslos

## Skalierung und Redundanz

### Mehrere Anwendungs-Instanzen

Wenn eine Anwendung skaliert wird (mehrere Container), teilen sich alle Container das gleiche Volume:

```
App Instance 1 ─┐
App Instance 2 ─┤─→ Shared Volume (z.B. lld-uploads)
App Instance 3 ─┘
```

Dies ermöglicht es, dass alle Instanzen die gleichen Daten sehen.

### Datenbank-Replikation (Zukunft)

In späteren Phasen könnte PostgreSQL repliziert werden:

```
PostgreSQL Primary ──→ Write Operations
    ↓
PostgreSQL Replica ──→ Read Operations (optional)
    │
    └─→ Shared Storage (für Failover)
```

Die Architektur ermöglicht dies, ohne grundlegend geändert zu werden.

## Langfristige Erweiterungen

### Objekt-Storage

Für sehr große Dateien oder Medien-Verwaltung:
- S3-kompatible Objekt-Speicher
- Unbegrenzte Skalierbarkeit
- Kosteneffizient für große Mengen

Die Architektur bleibt gleich — nur der Backend-Speicher ändert sich.

### Remote-Backups

Backups können zu Remote-Speichern repliziert werden:
- AWS S3, Azure Blob Storage
- Off-Site-Redundanz
- Disaster-Recovery

Dies schützt vor Datenmiete durch lokale Ausfälle.

### Weitere Storage-Klassen

Verschiedene Datentypen mit unterschiedlichen Anforderungen:
- **Hot Storage:** Aktiv genutzte Daten (schnell)
- **Warm Storage:** Gelegentlich genutzte Daten
- **Cold Storage:** Archivdaten (günstig)

Die Architektur unterstützt mehrere Storage-Klassen parallel.

### Skalierbare Datenhaltung

Wenn mehrere Kundenwebseiten existieren:
- Jeder Kunde hat eigene Volumes
- Isolation und Sicherheit
- Unabhängige Backups pro Kunde
- Skalierbarkeit auf beliebig viele Kunden

## Zusammenfassung

Die Storage-Architektur der LLD-Infrastruktur ist:

- **Persistent** — Daten überleben Neustarts
- **Wartbar** — Nachvollziehbare Struktur und Namenskonvention
- **Skalierbar** — Unterstützt Wachstum ohne Umbauten
- **Sicher** — Trennung von Container und Daten
- **Zukunftsorientiert** — Erweiterbar für neue Storage-Typen

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/networking.md](networking.md) — Netzwerkarchitektur
- [docs/architecture/deployment.md](deployment.md) — Deployment-Architektur
- [ROADMAP.md](../../ROADMAP.md) — Backup-und-Recovery-Phase
