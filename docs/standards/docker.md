# Docker-Standards — lld-infrastructure

Verbindliche Standards für den Einsatz von Docker innerhalb der LLD-Infrastruktur.

## Ziel des Docker-Standards

Die Docker-Standards definieren die verbindlichen Regeln für den Einsatz von Containern und Images im Projekt.

Ziele:

### Einheitlichkeit
Alle Container und Images folgen denselben Prinzipien. Dies ermöglicht einheitliche Wartbarkeit und Verständnis.

### Wartbarkeit
Standards machen die Infrastruktur wartbar. Neue Team-Mitglieder können schnell verstehen, wie Docker verwendet wird.

### Reproduzierbarkeit
Container und Images sind versioniert und reproduzierbar. Ein Deployment ist identisch zum vorherigen.

### Sicherheit
Standards schließen unsichere Praktiken aus (z.B. SSH in Produktionscontainer, manuelle Hotfixes).

## Grundprinzipien

### Ein Service pro Container

Jeder Container läuft **genau einen Service**:
- Ein Django-Container für die Panel-Anwendung
- Ein PostgreSQL-Container für die Datenbank
- Ein Nginx-Container für den Reverse Proxy

**Ausnahme:** Unterstützungs-Prozesse (z.B. Logging-Agenten) sind akzeptabel, wenn sie eng zum Hauptservice gehören.

**Konsequenz:** Dies macht Container einfach zu verstehen und zu warten.

### Container sind austauschbar

Ein Container ist eine **austauschbare Instanz** eines Images:
- Container A läuft nicht mehr → Container B startet mit dem gleichen Image
- Die Plattform funktioniert unverändert
- Daten sind erhalten (nicht im Container, sondern in Volumes)

**Konsequenz:** Container können zu jederzeit neu gestartet, ersetzt oder skaliert werden.

### Container speichern keine persistenten Daten

Container sind **ephemeral** (flüchtig). Sie sollten **keine** persistenten Daten enthalten.

Persistente Daten liegen in Docker Volumes, nicht im Container-Dateisystem.

**Konsequenz:** Container können ohne Datenverlust gelöscht und neu erzeugt werden.

### Container sind möglichst zustandslos

Container sollten **möglichst wenig Zustand** haben.

Ein Neustart sollte die Funktionalität nicht beeinträchtigen.

**Ausnahme:** Services wie PostgreSQL haben Zustand (Datenbank), dieser liegt aber in Volumes.

**Konsequenz:** Services sind robust gegen Neustarts und Fehler.

### Images werden versioniert

Jedes Image hat eine **eindeutige Version** (z.B. `v1.0.0`, `v1.1.0`).

Images werden nicht ohne Versionierung verwendet.

**Konsequenz:** Jeder Deployment ist reproduzierbar.

## Container-Richtlinien

### Klare Verantwortlichkeit pro Container

Jeder Container hat **genau eine, klar definierte Aufgabe**:

- Panel-Container: Panel-Anwendung ausführen
- Postgres-Container: Datenbank verwalten
- Nginx-Container: HTTP(S)-Routing

Die Aufgabe ist dokumentiert und unverrückbar.

**Konsequenz:** Container sind leicht zu verstehen, Fehler sind schnell lokalisiert.

### Keine manuellen Änderungen innerhalb laufender Container

Container werden **nicht interaktiv geändert**. Kein SSH, kein Exec, keine manuellen Befehle im laufenden Container.

Änderungen werden:
- Im Code/Image implementiert
- In Git versioniert
- Durch einen neuen Deployment ausgerollt

**Konsequenz:** Der Zustand ist versioniert und reproduzierbar, nicht verloren nach einem Neustart.

### Konfiguration erfolgt über Umgebungsvariablen

Container-Konfiguration wird durch Umgebungsvariablen übergeben, nicht durch Dateien im Container.

Beispiele:
- `DATABASE_URL` — Datenbankverbindung
- `DEBUG_MODE` — Debug-Modus an/aus
- `API_KEY` — Authentifizierung

**Konsequenz:** Konfiguration ist flexibel und umgebungsspezifisch (Development vs. Production).

### Container werden ausschließlich aus versionierten Images erzeugt

Container starten **niemals** aus `latest` oder unstabilen Tags.

Jeder Container startet aus einem **expliziten, versionierten Image**:
- `panel:v1.2.3`
- `postgres:v14.5`
- `nginx:v1.24.0`

**Konsequenz:** Deployments sind vorhersagbar und nicht vom Zufall abhängig.

## Image-Richtlinien

### Keine `latest`-Tags

Das `latest`-Tag ist in dieser Infrastruktur **verboten**.

`latest` ist unkontrolliert und ändert sich ständig — das ist das Gegenteil von Reproduzierbarkeit.

**Stattdessen:** Explizite Versionsnummern verwenden:
- `v1.0.0`
- `v1.1.0-alpha.1`
- `2024-06-27` (Datum-basiert)

**Konsequenz:** Jeder Deployment ist vorhersagbar.

### Versionierte Images

Eigene Images werden versioniert und getaggt.

Jedes Release bekommt ein Tag:
- `lld-panel:v0.1.0`
- `lld-website:v0.1.0`

**Konsequenz:** Images sind nachverfolgbar und reproduzierbar.

### Offizielle Images bevorzugen

Wenn möglich, werden **offizielle, gepflegte Images** von Docker Hub verwendet:
- `postgres:14.5-alpine`
- `python:3.11-slim`
- `node:18-alpine`

Nicht: zufällige Third-Party-Images

**Grund:** Offizielle Images sind gepflegt, sicher und gut dokumentiert.

**Konsequenz:** Höhere Sicherheit und Zuverlässigkeit.

### Eigene Images werden dokumentiert und versioniert

Wenn eigene Images notwendig sind:
- **Dokumentation:** Zweck und Inhalt des Images
- **Versionierung:** Explizite Version-Tags
- **Quellcode:** Dockerfile in diesem Repository gespeichert
- **Changelog:** Änderungen zwischen Versionen dokumentiert

**Konsequenz:** Eigene Images sind wartbar und nachvollziehbar.

## Persistenz

### Persistente Daten liegen außerhalb der Container

Container sind austauschbar. Persistente Daten sind nicht.

Daher: **Persistente Daten liegen in Docker Volumes**, nicht im Container-Dateisystem.

Beispiele:
- Datenbankdaten → PostgreSQL Volume
- Benutzer-Uploads → Media Volume
- Konfigurationen → Config Volume

**Konsequenz:** Daten überleben Container-Neustarts.

### Verwendung benannter Docker Volumes

Volumes haben **eindeutige Namen**, nicht generierte UUIDs:
- `lld-postgres-data`
- `lld-uploads`
- `lld-config`

Nicht: `volume_abc123` oder automatisch generierte Namen

**Konsequenz:** Volumes sind identifizierbar und wartbar.

### Keine anonymen Volumes

Anonyme Volumes (automatisch generiert, ohne Namen) sind **verboten**.

Alle Volumes müssen einen **klaren, eindeutigen Namen** haben.

**Grund:** Anonyme Volumes werden leicht "verloren" und sind unmöglich zu warten.

**Konsequenz:** Alle Volumes sind bekannt und gepflegt.

## Nicht erlaubt

Die folgenden Praktiken sind **explizit verboten** und dürfen nicht vorkommen:

### SSH in Produktionscontainer

SSH in laufende Produktionscontainer zur Änderung ist **nicht erlaubt**.

Dies umgeht Versionskontrolle und macht das System unvorhersehbar.

**Grund:** Der Zustand wäre nicht in Git, nicht reproduzierbar, nicht wartbar.

**Stattdessen:** Probleme werden behoben, indem Code/Konfiguration angepasst und neu deployiert wird.

### Manuelle Hotfixes

Schnelle, unversionierte Fixes in laufenden Systemen sind **nicht erlaubt**.

Jede Änderung geht durch:
1. Code-Anpassung
2. Git-Commit
3. Version-Tag
4. Neuer Deployment

**Grund:** Manuell gefixt ist ein Hotfix nicht reproduzierbar und wird bei nächstem Restart überschrieben.

**Konsequenz:** Alle Fixes sind versioniert und dokumentiert.

### Änderungen ohne Git

Alle Änderungen an der Infrastruktur müssen in Git versioniert sein.

Keine Out-of-Band-Änderungen, die nicht in Git dokumentiert sind.

**Grund:** Git ist Single Source of Truth.

**Konsequenz:** Der Zustand des Systems ist versioniert und reproduzierbar.

### Direkte Änderungen an laufenden Containern

Container werden **nicht während Laufzeit** konfiguriert.

Änderungen werden:
- Im Code/Compose gemacht
- In Git committed
- Durch Neuerstellung des Containers ausgerollt

**Grund:** Änderungen während Laufzeit sind flüchtig und gehen beim nächsten Neustart verloren.

**Konsequenz:** Alle Änderungen sind persistent und reproduzierbar.

## Compliance und Auditing

### Alle Docker-Operationen werden protokolliert

Jeder Start, Stop, Restart eines Containers wird protokolliert:
- Wann?
- Wer? (wenn manuell)
- Was?
- Warum?

Dies ist essentiell für Audit und Debugging.

### Regelmäßige Security-Scans

Images werden regelmäßig auf Sicherheitslücken gescannt:
- Veraltete Abhängigkeiten
- Bekannte CVEs
- Unsichere Konfigurationen

Updates werden geplant und durchgeführt.

## Langfristige Entwicklung

### Automatisierte Image-Builds

Zukünftig werden Images automatisiert erstellt:
- Bei Git-Pushes
- Mit automatischen Tests
- Mit automatischen Security-Scans
- Mit automatischem Tagging

Dies verbessert Sicherheit und Effizienz.

### Image-Registries

Eigene Images werden in privaten Registries gespeichert und versioniert:
- Zentrale Verwaltung
- Zugriffskontrolle
- Automatische Cleanup alter Versionen

### Multi-Stage Builds

Images werden optimiert durch Multi-Stage-Builds:
- Kleinere Dateigröße
- Schnellere Deployments
- Weniger Oberflächenangriff

### Container Runtime Security

Erweiterte Sicherheitsmechanismen:
- Network Policies
- Pod Security Policies
- Runtime Monitoring
- Anomaly Detection

## Zusammenfassung

Die Docker-Standards der LLD-Infrastruktur sind:

- **Einheitlich** — Alle Container folgen gleichen Prinzipien
- **Austauschbar** — Container sind reproduzierbar und ersetzbbar
- **Versioniert** — Images und Deployments sind eindeutig versioniert
- **Sicher** — Unsichere Praktiken sind explizit verboten
- **Wartbar** — Standards ermöglichen langfristige Pflege

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](../architecture/overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/deployment.md](../architecture/deployment.md) — Deployment-Architektur
- [docs/architecture/storage.md](../architecture/storage.md) — Storage-Architektur
