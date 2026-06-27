# Deployment-Architektur — lld-infrastructure

Konzeptionelle Dokumentation des Deployment-Prozesses für die LLD-Infrastruktur.

## Ziel des Deployment-Prozesses

Der Deployment-Prozess verfolgt folgende Ziele:

### Reproduzierbarkeit
Jeder Deployment ist identisch. Nicht von Umgebungsvariablen abhängig, nicht vom Zufall.

### Dokumentiert
Jeder Deployment ist nachvollziehbar. Was wurde geändert? Wann? Warum?

### Versionsbasiert
Deployments basieren auf expliziten Versionen, nicht auf "latest" oder unkontrollierten Branches.

### Nachvollziehbar
Es ist möglich, den Zustand des Systems zu jedem Zeitpunkt zu rekonstruieren. Auch nach Monaten.

### Wartbar
Der Deployment-Prozess selbst ist wartbar und erweiterbar. Neue Phasen können hinzugefügt werden, ohne existierende zu brechen.

## Deployment-Prinzipien

### Git ist die einzige Quelle der Wahrheit
Der Zustand aller Systeme (Infrastruktur und Anwendungen) ist ausschließlich in Git definiert.

Es gibt keine Out-of-Band-Änderungen. Keine manuellen Befehle auf dem Server, die nicht in Git versioniert sind.

### Keine manuellen Änderungen auf dem Server
Der Server ist nicht interaktiv zu ändern. Alles erfolgt durch Git-Operationen und definierte Deployment-Prozesse.

Dies verhindert:
- Menschliche Fehler
- Undokumentierte Zustände
- Drift zwischen Entwicklung und Produktion

### Infrastructure as Code
Alle Infrastruktur-Komponenten sind Code: Konfigurationen, Deployments, Rollbacks.

Code kann versioniert, überprüft und diskutiert werden.

### Trennung zwischen Anwendung und Infrastruktur
Anwendungs-Code (lld-panel, Kundenwebseiten) liegt in separaten Repositories.

Infrastruktur-Code liegt in diesem Repository.

Diese Trennung ermöglicht:
- Unabhängige Entwicklung
- Unterschiedliche Versions-Zyklen
- Modulare Verwendung

### Trennung zwischen Konfiguration und Quellcode
Umgebungsspezifische Konfiguration (z.B. Datenbank-Passwörter) ist getrennt vom Code.

Quellcode ist austauschbar. Konfiguration ist umgebungsspezifisch.

### Rollbacks erfolgen über Versionen
Um auf einen vorherigen Zustand zurückzukehren, wird zu einer vorherigen Version gewechselt.

Nicht durch "rückgängig machen", sondern durch explizite Versions-Verwaltung.

## Deployment-Flow

Der logische Ablauf vom Quellcode bis zum laufenden Service:

```
Developer
    ↓
Git Repository
    ↓
Server (Infrastruktur-Repository ausgecheckt)
    ↓
Docker Compose (Orchestrierung)
    ↓
Container (Anwendungen starten)
    ↓
Running Application
    ├── panel.lukasleimer.dev
    ├── lukasleimer.dev
    └── Kundenwebseiten
```

**Konzeptionell:**

1. Ein Developer committed Code in ein Git Repository (Anwendung oder Infrastruktur)
2. Dieser Code wird auf den Server gezogen
3. Die Infrastruktur interpretiert die neuen Definitionen
4. Docker Compose orchestriert die neuen Container
5. Container starten oder werden aktualisiert
6. Services sind verfügbar

Dieser Fluss ist:
- Linear und nachvollziehbar
- Nicht losgelöst von Git
- Wiederholbar und testbar

## Deployment-Phasen

Ein Deployment durchläuft folgende konzeptionelle Phasen:

### Phase 1: Entwicklung
Code wird in einem Develop-Environment entwickelt und getestet.

Dies kann lokal sein oder in einer Development-Umgebung.

### Phase 2: Versionierung
Code wird committed und in Git versioniert.

Explizite Versionen werden getaggt (z.B. `v0.1.0`).

Der Zustand ist reproduzierbar.

### Phase 3: Deployment
Die neue Version wird auf dem Zielserver ausgecheckt.

Infrastruktur-Definitionen werden eingelesen.

Dies kann automatisiert oder manuell erfolgen (siehe: Langfristige Entwicklung).

### Phase 4: Service-Start
Services werden in neuen oder aktualisierten Containern gestartet.

Abhängigkeiten werden berücksichtigt (z.B. Datenbank vor Anwendung).

### Phase 5: Verifikation
Das System wird überprüft:
- Services sind erreichbar
- Health Checks bestätigen Funktionalität
- Fehler werden erkannt und kommuniziert

## Verantwortlichkeiten

### Git
- Speichert den Quellcode und die Infrastruktur-Definitionen
- Ist Versionsverwaltung und Single Source of Truth
- Kommuniziert Änderungen zwischen Teams

### lld-infrastructure Repository
- Definiert alle Infrastruktur-Komponenten
- Docker-Container und deren Abhängigkeiten
- Netzwerke, Volumes, Konfigurationen
- Deployment-Prozesse und Rollbacks

### Anwendungs-Repositories (lld-panel, Kundenwebseiten)
- Enthalten Applikationscode
- Definieren ihre Anforderungen an die Infrastruktur
- Sind unabhängig von der Infrastruktur-Implementierung

### Docker
- Isoliert Anwendungen in Containern
- Garantiert Konsistenz zwischen Umgebungen
- Ermöglicht reproduzierbare Deployments

### Ubuntu Server
- Führt die Infrastruktur-Definitionen aus
- Startet und verwaltet Container
- Stellt Ressourcen (CPU, RAM, Speicher) bereit
- Überwacht die Gesundheit der Services

## Langfristige Entwicklung

Der Deployment-Prozess wird schrittweise ausgereift:

### Phase 1: Manuelles Deployment
**Zeitraum:** Foundation bis Docker Compose

Deployment erfolgt durch `git pull` und manuelle Kommandos.

Developer zieht die neueste Version und startet Services.

**Vorteile:**
- Einfach zu verstehen
- Explizit und kontrolliert
- Keine zusätzliche Automation nötig

**Nachteile:**
- Anfällig für menschliche Fehler
- Zeitaufwändig
- Nicht automatisierbar

### Phase 2: Teilautomatisierte Deployments
**Zeitraum:** Post Docker Compose bis Monitoring

Deployment-Scripts automatisieren häufige Operationen.

CI/CD-Pipeline beginnt Deployments zu unterstützen (z.B. GitHub Actions).

**Vorteile:**
- Konsistente Deployments
- Automatisierte Tests
- Schneller und zuverlässiger

**Nachteile:**
- Zusätzliche Komplexität
- Anforderung an CI/CD-System

### Phase 3: Deployment über LLD Panel
**Zeitraum:** Nach LLD Panel Integration

Deployments erfolgen über das Admin-Panel.

Buttons oder automatische Trigger (z.B. bei neuen Releases).

Ein-Click-Deployments möglich.

**Vorteile:**
- Benutzerfreundlich
- Niedrige Einstiegshürde
- Zentrale Verwaltung

**Nachteile:**
- Höchste Komplexität
- Abhängigkeit vom Panel
- Muss sicher sein

## Zusammenfassung

Die Deployment-Architektur der LLD-Infrastruktur ist:

- **Einfach** — Der Prozess ist nachvollziehbar und linear
- **Sicher** — Git ist Single Source of Truth, keine Workarounds
- **Wartbar** — Der Prozess selbst ist dokumentiert und erweiterbar
- **Zukunftsorientiert** — Evolutionäre Verbesserung, nicht Revolution

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Verwandte Dokumente
- [docs/architecture/overview.md](overview.md) — Allgemeine Architekturübersicht
- [ROADMAP.md](../../ROADMAP.md) — Entwicklungsplanung und Phasen
