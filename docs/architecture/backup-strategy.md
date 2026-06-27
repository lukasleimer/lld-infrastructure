# Backup- und Restore-Architektur — lld-infrastructure

Konzeptionelle Dokumentation der Datensicherungs- und Wiederherstellungsstrategie der LLD-Infrastruktur.

## Ziel der Backup-Strategie

Die Backup-Strategie verfolgt folgende Ziele:

### Schutz vor Datenverlust

Daten sind der wertvollste Bestand einer Infrastruktur. Backups schützen vor:
- Hardware-Ausfällen
- Menschlichen Fehlern
- Bösartigen Angriffen
- Naturkatastrophen

### Wiederherstellbarkeit

Im Schadensfall kann die Plattform aus gesicherten Daten wiederhergestellt werden.

Der Punkt-in-Zeit kann zurück zu einem früheren, bekannten Zustand.

### Nachvollziehbarkeit

Backups werden versioniert und dokumentiert:
- Wann wurde gesichert?
- Was wurde gesichert?
- Kann das Backup wiederhergestellt werden?

Dies ermöglicht Kontrolle und Compliance.

### Zuverlässigkeit

Backups sind nicht optional. Sie sind verpflichtender Bestandteil der Plattform.

Backup-Fehlschläge werden erkannt und kommuniziert.

### Reproduzierbarkeit

Backups ermöglichen es, die Plattform zu **jedem** früheren Zeitpunkt exakt zu reproduzieren.

Dies ist essentiell für Infrastructure as Code.

## Backup-Grundsätze

### Backups sind verpflichtender Bestandteil

Backups sind nicht optional oder nachgedacht. Sie sind zentral und ständig aktiv.

Jede Produktions-Umgebung hat kontinuierliche Backups.

### Backups werden versioniert

Jedes Backup hat:
- Eindeutige ID (Timestamp oder UUID)
- Eindeutige Version (v1, v2, v3, etc.)
- Eindeutige Signatur (Checksum)

Dies ermöglicht es, spezifische Backups zu identifizieren und zu restore.

### Restore-Prozesse werden regelmäßig überprüft

Ein Backup ist nur so wertvoll wie seine Wiederherstellbarkeit.

Regelmäßige Recovery-Tests validieren:
- Backups sind nicht beschädigt
- Restore funktioniert
- RTO (Recovery Time Objective) ist eingehalten

### Backups sind unabhängig von Containern

Backups laufen **außerhalb** der Plattform und sind nicht abhängig von Running Containers.

Wenn alle Container ausfallen, können Backups trotzdem wiederhergestellt werden.

Dies wird durch dedizierte Backup-Prozesse und -Speicherung erreicht.

### Backup und Restore werden dokumentiert

Jeder Backup- und Restore-Prozess wird dokumentiert:
- Was wurde gesichert?
- Wann?
- Wie lange dauerte es?
- War es erfolgreich?
- Wer führte es aus?

Diese Dokumentation ist essentiell für Audit und Compliance.

## Zu sichernde Daten

### PostgreSQL-Datenbanken

**Bedeutung:** Strukturierte Daten der gesamten Plattform (Benutzer, Inhalte, Metadaten)

**Kritikalität:** 🔴 Kritisch — Datenverlust ist inakzeptabel

**Backup-Typ:** Konsistente Snapshots oder WAL-basierte Backup

### Medien und Uploads

**Bedeutung:** Benutzerhochladene Dateien (Bilder, PDFs, Kundenmedien)

**Kritikalität:** 🔴 Kritisch — Kundeninhalte dürfen nicht verloren gehen

**Backup-Typ:** File-System-basierte Backups (Kompression optional)

### Konfiguration

**Bedeutung:** Umgebungsspezifische Konfigurationen, API-Keys, Secrets

**Kritikalität:** 🟡 Hoch — Ohne Konfiguration läuft die Plattform nicht

**Backup-Typ:** Verschlüsselte Backups oder äußerst sichere Lagerung

### Persistente Volumes

**Bedeutung:** Alle Docker Volumes der Plattform

**Kritikalität:** 🔴 Kritisch — Alles kritische liegt in Volumes

**Backup-Typ:** Volume-Snapshots oder direktes Backup

### SSL-Zertifikate

**Bedeutung:** TLS-Zertifikate für alle Domains

**Kritikalität:** 🟡 Hoch — Ohne Zertifikate ist HTTPS nicht möglich

**Backup-Typ:** Verschlüsselte oder sichere Backups

### Infrastruktur-Konfiguration

**Bedeutung:** Docker Compose Dateien, Deployment-Scripts, Konfigurationen

**Kritikalität:** 🟡 Mittel — Liegt in Git, aber sollte auch separat gesichert werden

**Backup-Typ:** Git-Repository-Backups oder Snapshots

## Nicht zu sichernde Daten

### Container

Container sind **ephemeral**. Sie werden nicht gesichert.

Stattdessen wird die **Container-Definition** (Image, Konfiguration) in Git gespeichert.

Container können zu jederzeit neu aufgebaut werden.

### Images

Docker Images sind **rekonstruierbar** aus Dockerfiles.

Sie werden nicht gesichert — stattdessen wird die **Quelle** (Dockerfile) gesichert.

### Temporäre Daten

Caches, Session-Daten und Temp-Dateien sind **nicht kritisch**.

Sie können verloren gehen. Der Container wird neu gestartet und regeneriert sie.

### Build-Artefakte

Build-Ergebnisse sind **rekonstruierbar** aus Quellcode.

Sie werden nicht gesichert.

### Logs

Logs sind **informativ**, nicht kritisch für die Funktionalität.

Zu groß und zu häufig aktualisiert, um in Backups zu landen.

Separate Logging-Archivierung ist sinnvoll, aber nicht als Backup.

## Architekturübersicht

Der logische Ablauf von laufenden Diensten über Datensicherung bis zur Wiederherstellung:

```
Laufende Plattform
    │
    ├── PostgreSQL DB
    ├── User Uploads (Volumes)
    ├── Konfiguration
    └── SSL Certificates
    │
    ↓ (Kontinuierlich oder geplant)
    │
Backup-Prozess
    │
    ├→ DB Snapshot / Dump
    ├→ Volume Backup
    ├→ Config Backup
    └→ Zertifikat Backup
    │
    ↓ (Versioniert)
    │
Backup-Speicherung (Versionen)
    │
    ├── Backup v1 (2026-01-01 12:00)
    ├── Backup v2 (2026-01-02 12:00)
    ├── Backup v3 (2026-01-03 12:00)
    └── Backup vN (...)
    │
    ↓ (Im Notfall)
    │
Restore-Prozess
    │
    ├→ DB aus Backup wiederherstellen
    ├→ Volumes aus Backup wiederherstellen
    ├→ Konfiguration aus Backup wiederherstellen
    └→ Zertifikate aus Backup wiederherstellen
    │
    ↓
    │
Wiederhergestellte Plattform
    │
    └── Identisch zum Backup-Zeitpunkt
```

**Wichtig:**
- Backups sind **zeitlich** (Versionen nach Zeit)
- Backups sind **dezentralisiert** (nicht in der Plattform gespeichert)
- Restore ist **reproduzierbar** (immer das gleiche Ergebnis)

## Restore-Prinzipien

### Vollständige Wiederherstellung

Der gesamte Zustand der Plattform wird zu einem Backup-Zeitpunkt wiederhergestellt:

```
Plattform zu Zeitpunkt T
    ↓
[Disaster / Problem]
    ↓
Plattform zu Zeitpunkt T-N (aus Backup)
    ↓
Alles ist wie zu Zeitpunkt T-N
```

Dies ist das **Worst-Case-Szenario** — die komplette Plattform ist weg.

### Teilweise Wiederherstellung

Ein Teil der Plattform wird wiederhergestellt, anderes bleibt erhalten:

```
Plattform zu Zeitpunkt T
    ↓
[Kundenwebseite A beschädigt]
    ↓
Kundenwebseite A aus Backup T-N wiederherstellen
    ↓
Kundenwebseite A ist wiederhergestellt
[Kundenwebseite B und Panel laufen unverändert weiter]
```

Dies ist das **Normale Szenario** — spezifische Komponenten haben Probleme.

### Versionierte Backups

Jedes Backup hat eine **eindeutige Version**, so dass es möglich ist:
- Zu jedem Zeitpunkt zurückzugehen
- Zu verschiedenen Versionen zu vergleichen
- Zu entscheiden, welche Version wiederhergestellt wird

### Dokumentierte Restore-Prozesse

Jeder Restore-Prozess ist dokumentiert:
- Schritt-für-Schritt Anweisungen
- Fehlerbehebung
- Validierung nach Restore
- Zeitschätzungen

Damit nicht nur Techniker, sondern auch Nicht-Techniker verstehen, wie es funktioniert.

### Reproduzierbare Plattform

Nach einem Restore ist die Plattform **identisch** zum Backup-Zeitpunkt:
- Alle Daten sind gleich
- Alle Konfigurationen sind gleich
- Alle Services funktionieren gleich

Dies ist Infrastructure as Code in Aktion.

## Backup-Strategie: 3-2-1 Regel

Die bewährte **3-2-1 Backup-Regel** wird angewendet:

### 3 Kopien der Daten

Mindestens 3 Kopien existieren zu jeder Zeit:
1. **Original** — Die produktive Plattform
2. **Backup 1** — Lokales Backup (schneller Zugriff)
3. **Backup 2** — Remote-Backup (Offsite-Schutz)

### 2 Verschiedene Speichermedien

Backups liegen auf verschiedenen Medien:
- Lokal: Filesystem / SSD
- Remote: Cloud-Storage oder externe Server

Dies schützt vor Medienfehlern.

### 1 Offsite-Kopie

Mindestens eine Kopie liegt an einem anderen Standort:
- Remote Cloud-Storage
- Externer Server
- Physisch anderer Standort

Dies schützt vor physischen Katastrophen.

## Langfristige Erweiterungen

### Remote-Backups

Backups werden zu Remote-Speichern repliziert:
- AWS S3, Azure Blob Storage, etc.
- Automatische Sync
- Geografische Redundanz
- Langzeitarchivierung

### Verschlüsselte Backups

Sensitive Daten (Datenbanken, Konfiguration) werden verschlüsselt:
- End-to-End-Verschlüsselung
- Schlüsselverwaltung
- Offsite-Schlüsselspeicherung

### Automatisierte Backup-Prüfungen

Regelmäßige automatisierte Tests:
- Können Backups wiederhergestellt werden?
- Sind Backups beschädigt?
- Wie lange dauert Restore?
- Sind RTO/RPO erfüllt?

Diese Tests laufen automatisiert und melden Fehler.

### Mehrere Backup-Ziele

Verschiedene Backup-Szenarien für verschiedene Anforderungen:
- **Minutenbasiert** — Minimales RPO (Recovery Point Objective)
- **Täglich** — Standard-Backups
- **Wöchentlich** — Langzeitarchivierung
- **Monatlich** — Compliance und Audit

### Disaster-Recovery-Konzepte

Erweiterte DR-Strategien:
- **RTO** (Recovery Time Objective) — Wie schnell muss Plattform wieder laufen?
- **RPO** (Recovery Point Objective) — Wie viel Datenverlust ist akzeptabel?
- **Failover-Sites** — Redundante Infrastruktur in anderer Region
- **Disaster-Recovery-Drills** — Regelmäßige Praxistests

## Zusammenfassung

Die Backup-Strategie der LLD-Infrastruktur ist:

- **Verpflichtend** — Kontinuierliche, automatisierte Backups
- **Versioniert** — Jedes Backup hat eindeutige Identität
- **Getestet** — Restore-Prozesse werden regelmäßig validiert
- **Dokumentiert** — Jeder Prozess ist nachvollziehbar
- **Zukunftsorientiert** — Erweiterbar für erweiterte Szenarien

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/storage.md](storage.md) — Storage-Architektur
- [ROADMAP.md](../../ROADMAP.md) — Backup & Recovery Phase
