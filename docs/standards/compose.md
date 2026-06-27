# Docker Compose Standards — lld-infrastructure

Verbindliche Standards für Docker Compose innerhalb der LLD-Infrastruktur.

## Ziel des Compose-Standards

Die Compose-Standards definieren die verbindlichen Regeln für die Struktur und Organisation von Docker Compose-Dateien.

Ziele:

### Einheitliche Struktur

Alle Compose-Dateien folgen den gleichen Organisationsprinzipien. Dies ermöglicht schnelles Verständnis und Konsistenz.

### Lesbarkeit

Compose-Dateien sind selbstdokumentierend und leicht verständlich für Entwickler und Operateure.

### Modularität

Services und Ressourcen sind modular organisiert. Neue Services können leicht hinzugefügt werden, ohne Bestehendes zu beeinflussen.

### Wartbarkeit

Die Strukturen sind wartbar und erweiterbar für langfristige Entwicklung.

### Erweiterbarkeit

Neue Services, Netzwerke und Volumes können hinzugefügt werden, ohne fundamentale Umbauten.

## Grundprinzipien

### Docker Compose ist der Standard zur Service-Orchestrierung

Docker Compose ist das zentrale Tool zur Orchestrierung aller Services der LLD-Plattform.

**Alternativtechnologien** (Kubernetes, Swarm) werden **nicht** verwendet.

**Grund:** Einfachheit und Wartbarkeit für Single-Server-Setup.

**Konsequenz:** Alle Services werden über Compose orchestriert.

### Eine Compose-Datei beschreibt einen logischen Infrastruktur-Stack

Jede Compose-Datei definiert einen **logisch zusammenhängenden Stack** von Services:
- **main.compose.yml** — Kern-Services (Reverse Proxy, PostgreSQL, Panel, Website)
- **monitoring.compose.yml** — Monitoring-Services (zukünftig)
- **backup.compose.yml** — Backup-Services (zukünftig)

Nicht: Eine monolithische Datei mit allem vermischt.

**Konsequenz:** Services sind logisch organisiert und wartbar.

### Dienste werden modular organisiert

Services sind unabhängig voneinander definiert und können unabhängig skaliert oder ausgetauscht werden.

Abhängigkeiten zwischen Services sind explizit dokumentiert.

**Konsequenz:** Services sind wartbar und erweiterbar.

### Die Konfiguration bleibt deklarativ

Compose-Dateien definieren den **Soll-Zustand**, nicht den Prozess.

Die Datei beschreibt: "Dieser Service soll mit diesen Eigenschaften laufen."

Nicht: "Führe diese Schritte aus, um den Service zu starten."

**Konsequenz:** Deployments sind reproducierbar und verständlich.

### Laufende Services werden nicht manuell verändert

Services werden nicht durch direkte Befehle verändert, sondern durch Anpassung der Compose-Datei und Neudeployment.

**Grund:** Der Zustand muss in der Datei dokumentiert sein.

**Konsequenz:** Der Zustand ist versioniert und reproduzierbar.

## Service-Richtlinien

### Eindeutige Servicenamen

Jeder Service hat einen **eindeutigen, aussagekräftigen Namen**:
- `reverse-proxy` — Reverse Proxy Service
- `postgres` — Datenbank
- `panel` — Admin-Panel
- `website` — Unternehmenswebseite

Nicht: `service1`, `app`, `db` (mehrdeutig)

**Konsequenz:** Services sind identifizierbar und nachvollziehbar.

### Eine Verantwortung pro Service

Jeder Service hat **genau eine, klar definierte Aufgabe**:
- Der `reverse-proxy`-Service routet HTTP(S)-Traffic
- Der `postgres`-Service verwaltet die Datenbank
- Der `panel`-Service führt die Panel-Anwendung aus

Nicht: Ein Service, der mehrere Aufgaben hat.

**Konsequenz:** Services sind leicht zu verstehen und zu warten.

### Nutzung gemeinsamer Netzwerke

Alle Plattform-Services verbinden sich mit dem gemeinsamen Docker-Netzwerk `lld`:
- Services können sich über Servicenamen ansprechen
- Netzwerk ist zentral verwaltet
- Kommunikation ist nachvollziehbar

**Konsequenz:** Services sind im selben Netzwerk, können sich aber isolieren.

### Nutzung benannter Volumes

Services speichern persistente Daten in **benannten Volumes**, nicht in anonymen Volumes:
- `lld-postgres-data` — PostgreSQL-Daten
- `lld-uploads` — Benutzerdateien
- `lld-config` — Konfiguration

Nicht: Automatisch generierte Volume-Namen

**Konsequenz:** Volumes sind identifizierbar und wartbar.

### Healthchecks für produktive Dienste

Services in Production haben **Healthchecks**:
- Prüfung, ob Service erreichbar ist
- Automatische Neustarts bei Fehler
- Monitoring ermöglicht

**Beispiel:** Ein `panel`-Service prüft seine HTTP-Schnittstelle

Nicht: Services ohne Healthchecks

**Konsequenz:** Die Plattform ist robust gegen Fehler.

## Netzwerke und Volumes

### Gemeinsames Docker-Netzwerk `lld`

Alle Plattform-Services verwenden das Netzwerk `lld`:
- Service-Discovery durch DNS (Servicenamen)
- Isolierung von außen
- Zentrale Verwaltung

Das Netzwerk ist in der Compose-Datei **explizit definiert**, nicht implizit.

**Konsequenz:** Netzwerk-Struktur ist dokumentiert.

### Benannte Docker Volumes

Alle Volumes haben **eindeutige, aussagekräftige Namen**:
- `lld-postgres-data`
- `lld-uploads`
- `lld-config`
- `customer-001-data`

Nicht: `volume_abc123`, `app_data`, oder automatisch generierte Namen

**Namenskonvention:** `{project}-{purpose}` oder `{customer}-{data}`

**Konsequenz:** Volumes sind identifizierbar und wartbar.

### Zentrale Verwaltung

Netzwerke und Volumes werden in der Compose-Datei **zentral definiert**, nicht implizit durch Services erzeugt.

Dies ermöglicht einen Überblick über alle Ressourcen.

**Konsequenz:** Ressourcen sind dokumentiert und verwaltet.

### Keine anonymen Ressourcen

Anonyme (automatisch generierte) Netzwerke und Volumes sind **nicht erlaubt**.

Alle Ressourcen haben explizite Namen in der Compose-Datei.

**Grund:** Anonyme Ressourcen können verloren oder vergessen gehen.

**Konsequenz:** Alle Ressourcen sind nachvollziehbar.

## Wartbarkeit

### Lesbarkeit

Compose-Dateien sind **selbstdokumentierend**:
- Logische Gruppierung von Services
- Aussagekräftige Namen
- Kommentare an kritischen Stellen
- Konsistente Struktur

**Konsequenz:** Neue Team-Mitglieder verstehen schnell.

### Erweiterbarkeit

Neue Services können leicht hinzugefügt werden:
- Keine Abhängigkeiten zwischen Services müssen geändert werden
- Netzwerk und Volumes sind modular
- Struktur bleibt konsistent

**Konsequenz:** Neue Services haben minimale Reibung.

### Wiederverwendbarkeit

Compose-Dateien oder Services können als Templates für ähnliche Setups verwendet werden:
- Kundenwebseiten-Template
- Monitoring-Services
- Backup-Services

**Konsequenz:** Weniger Duplikation, mehr Konsistenz.

### Dokumentation

Jede Compose-Datei wird dokumentiert:
- **Zweck:** Was beschreibt diese Datei?
- **Services:** Welche Services sind enthalten?
- **Abhängigkeiten:** Welche anderen Stacks sind notwendig?
- **Konfiguration:** Welche Umgebungsvariablen sind erforderlich?

**Konsequenz:** Compose-Dateien sind selbsterklärend.

## Nicht erlaubt

Die folgenden Praktiken sind **explizit verboten**:

### Monolithische Compose-Dateien ohne Struktur

Eine riesige `docker-compose.yml` mit allen Services vermischt ist **nicht erlaubt**.

Stattdessen: Modulare Struktur mit mehreren Dateien.

**Grund:** Monolithen sind unlesbar und unmöglich zu warten.

**Konsequenz:** Compose-Dateien sind klein und fokussiert.

### Anonyme Volumes

Volumes ohne eindeutige Namen sind **nicht erlaubt**:

Nicht: `- /data` (anonym)
Stattdessen: `- lld-postgres-data:/data` (benannt)

**Grund:** Anonyme Volumes werden verloren.

**Konsequenz:** Alle Volumes sind nachverfolgbar.

### Nicht dokumentierte Services

Jeder Service in einer Compose-Datei muss dokumentiert sein:
- Zweck des Services
- Abhängigkeiten
- Erforderliche Umgebungsvariablen
- Health-Status

Undokumentierte Services sind **nicht erlaubt**.

**Grund:** Ohne Dokumentation können Services nicht gewartet werden.

**Konsequenz:** Alle Services sind verständlich.

### Direkte Änderungen an laufenden Deployments

Services werden nicht durch direkte Docker-Befehle verändert.

Alle Änderungen erfolgen durch:
1. Änderung der Compose-Datei
2. Commit in Git
3. Neudeployment aus der aktualisierten Datei

**Grund:** Direkte Änderungen sind nicht versioniert und gehen beim nächsten Deployment verloren.

**Konsequenz:** Der Zustand ist versioniert.

### Abhängigkeiten außerhalb der Datei

Compose-Dateien dürfen **nicht von anderen Systemen abhängen**, die nicht dokumentiert sind.

Alle Abhängigkeiten müssen:
- In der Datei dokumentiert sein
- Oder als separate Compose-Dateien moduliert sein

**Grund:** Versteckte Abhängigkeiten führen zu Überraschungen und Fehlern.

**Konsequenz:** Abhängigkeiten sind explizit und verwaltet.

## Governance und Versioning

### Compose-Dateien sind versioniert

Änderungen an Compose-Dateien werden in Git committed:
- Jede Änderung wird dokumentiert
- Rollbacks sind möglich
- Historische Versionen sind erhalten

**Konsequenz:** Infrastruktur-Änderungen sind nachverfolgbar.

### Änderungen erfordern Review

Änderungen an Compose-Dateien werden vor Deployment überprüft:
- Code-Review
- Test in Entwicklungsumgebung
- Dokumentation der Änderung

Nicht: Sofortige Deployments ohne Überprüfung

**Konsequenz:** Fehler werden früh erkannt.

## Langfristige Entwicklung

### Kundenprojekte

Kundenwebseiten werden als **separate Compose-Dateien oder Stacks** organisiert:
- `customer-001.compose.yml`
- `customer-002.compose.yml`

Dies ermöglicht isolierte Verwaltung pro Kunde.

**Konsequenz:** Jeder Kunde kann unabhängig skaliert/verwaltet werden.

### Monitoring und Logging

Monitoring-Services werden in einer separaten Compose-Datei organisiert:
- `monitoring.compose.yml` — Prometheus, Grafana, etc.
- `logging.compose.yml` — Loki, ELK-Stack, etc.

Dies ermöglicht optionale Aktivierung/Deaktivierung.

**Konsequenz:** Monitoring ist modular integriert.

### Backup-Services

Backup-Services werden in einer separaten Compose-Datei organisiert:
- `backup.compose.yml`

Dies ermöglicht regelmäßige Backup-Operationen ohne Auswirkung auf Produktions-Services.

**Konsequenz:** Backups sind isoliert und zuverlässig.

### Automatisierte Deployments

In späteren Phasen werden Compose-Dateien durch Deployment-Prozesse automatisch orchestriert:
- GitHub Actions oder ähnliche CI/CD
- Automatische Tests
- Automatische Deployments

Compose-Dateien bleiben die zentrale Quelle.

**Konsequenz:** Deployments sind automatisiert und reproduzierbar.

### Template-Systeme

Häufig verwendete Compose-Strukturen werden als Templates abstrahiert:
- Customer-Template
- Service-Template
- Stack-Template

Dies reduziert Duplikation und Fehler.

**Konsequenz:** Neue Services/Kunden können schnell Setup-t werden.

## Zusammenfassung

Die Docker Compose Standards der LLD-Infrastruktur sind:

- **Modular** — Compose-Dateien sind logisch organisiert
- **Wartbar** — Services sind dokumentiert und nachvollziehbar
- **Einheitlich** — Alle Dateien folgen gleichen Strukturprinzipien
- **Erweiterbar** — Neue Services können leicht hinzugefügt werden
- **Versioniert** — Änderungen sind nachverfolgbar

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/standards/docker.md](docker.md) — Docker Standards
- [docs/architecture/overview.md](../architecture/overview.md) — Allgemeine Architekturübersicht
- [docs/architecture/deployment.md](../architecture/deployment.md) — Deployment-Architektur
