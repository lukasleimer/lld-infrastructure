# Security-Standards — lld-infrastructure

Verbindliche Sicherheitsstandards für die LLD-Infrastruktur.

## Ziel des Security-Standards

Die Security-Standards definieren die verbindlichen Anforderungen für Sicherheit in der LLD-Infrastruktur.

Ziele:

### Vertraulichkeit

Sensible Daten (Secrets, Passwörter, Benutzerinformationen) sind geschützt und nicht öffentlich zugänglich.

Nur autorisierte Systeme können auf diese Daten zugreifen.

### Integrität

Daten und Systeme werden nicht unbefugt verändert.

Änderungen sind nachverfolgbar und werden validiert.

### Verfügbarkeit

Die Plattform ist widerstandsfähig gegen Angriffe und Ausfälle.

Services sind erreichbar und funktional für berechtigte Nutzer.

### Nachvollziehbarkeit

Sicherheitsrelevante Ereignisse werden protokolliert und können später überprüft werden.

Im Schadensfall können Ursachen ermittelt werden.

## Grundprinzipien

### Least Privilege

Jede Komponente, jeder Benutzer und jedes System erhält **nur die minimalen Rechte**, die notwendig sind.

**Beispiele:**
- Services sprechen nur mit den Services, die nötig sind
- Datenbank-Benutzer haben nur auf ihre Datenbank Zugriff
- Entwickler haben Zugriff auf Development, nicht auf Production

**Konsequenz:** Falls eine Komponente kompromittiert wird, ist das Schadensausmaß minimal

### Defense in Depth

Sicherheit erfolgt nicht durch einen einzelnen Mechanismus, sondern durch **mehrere Schichten**.

**Beispiele:**
- Netzwerk-Firewall
- Container-Isolation
- Anwendungs-Level-Autorisierung
- Datenbank-Zugriffskontrolle

**Konsequenz:** Wenn eine Schicht durchbrochen wird, schützen andere Schichten

### Secure by Default

Systeme sind **sicher konfiguriert**, nicht erst nach zusätzlichen Konfigurationen.

**Nicht:** "SSL ist optional"
**Stattdessen:** "SSL ist aktiviert, kann nicht deaktiviert werden"

**Konsequenz:** Nicht konfigurierte Systeme sind automatisch sicher

### Principle of Least Exposure

Nur notwendige Funktionen und Schnittstellen sind öffentlich zugänglich.

Alles andere ist intern und nicht von außen erreichbar.

**Beispiel:** Reverse Proxy ist öffentlich, Datenbank nicht

**Konsequenz:** Angriffsfläche wird minimiert

### Security by Design

Sicherheit ist nicht nachgelagert, sondern **von Anfang an** eingebaut.

Architektonische Entscheidungen beachten Sicherheitsaspekte.

**Nicht:** "Wir sichern das später"
**Stattdessen:** "Architektur ist von Anfang an sicher"

**Konsequenz:** Teuer und kompliziert, aber fundamental sicher

## Zugriffskontrolle

### Nur notwendige öffentliche Dienste

Nur Services, die von außen erreichbar sein **müssen**, sind öffentlich:
- Reverse Proxy (HTTP/HTTPS-Gateway)

Das ist es. Alles andere ist intern.

**Nicht öffentlich:**
- Datenbanken
- Admin-Interfaces (sofern nicht im Panel integriert)
- Interne APIs
- Monitoring-Systeme

**Konsequenz:** Minimale Angriffsfläche

### Interne Dienste bleiben intern

Services im Docker-Netzwerk sind **nicht von außen erreichbar**:
- PostgreSQL läuft im `lld`-Netzwerk, nicht extern
- Services sprechen über interne Servicenamen, nicht über öffentliche IPs
- Keine Port-Mappings nach außen (außer Reverse Proxy)

**Konsequenz:** Interne Dienste sind geschützt

### Zugriff ausschließlich über definierte Schnittstellen

Zugriff auf Services erfolgt nur über **explizit definierte Schnittstellen**:
- Reverse Proxy definiert welche Domains wohin geroutet werden
- Nur diese Routen sind möglich
- Andere Zugriffe werden blockiert

**Nicht:** Direkter Zugriff auf Services von überall

**Konsequenz:** Zugriff ist kontrolliert und nachvollziehbar

### Klare Verantwortlichkeiten

Jeder Service / jede Komponente hat klare Verantwortlichkeiten für Sicherheit:
- **Reverse Proxy:** SSL-Terminierung, Request-Validation
- **Anwendungen:** Authentifizierung, Autorisierung
- **Datenbank:** Zugriffskontrolle, Datenverschlüsselung
- **Netzwerk:** Isolation, Segmentierung

Keine versteckten Verantwortlichkeiten.

**Konsequenz:** Sicherheitslücken sind schnell erkannt

## Secrets und Vertraulichkeit

### Keine Secrets im Repository

Passwörter, API-Keys und andere Secrets werden **niemals** in Git versioniert:
- Sie können nicht im Repository liegen
- Sie werden ausschließlich als Umgebungsvariablen bereitgestellt
- `.env.example` dokumentiert die erforderlichen Variablen, enthält aber keine echten Werte

**Siehe:** [docs/standards/environments.md](environments.md) für Details

**Konsequenz:** Repository kann öffentlich sein, ohne Sicherheitsrisiko

### Schutz sensibler Daten

Sensible Daten werden geschützt:
- Secrets werden verschlüsselt gespeichert
- Zugriff auf Secrets ist kontrolliert und protokolliert
- Nur autorisierte Systeme/Menschen haben Zugriff

**Beispiele sensibler Daten:**
- Datenbank-Passwörter
- API-Keys
- Private Keys für Zertifikate
- Session-Keys

### Regelmäßige Rotation

Secrets werden regelmäßig geändert:
- Passwörter: Alle 3-6 Monate
- API-Keys: Bei Verdacht auf Kompromittierung
- Session-Keys: Regelmäßig automatisiert

**Grund:** Reduziert Schadensrisiko bei Kompromittierung

**Konsequenz:** Langfristige Sicherheit

## Netzwerk

### Minimale Angriffsfläche

Das Netzwerk ist so strukturiert, dass die Angriffsfläche minimal ist:
- Nur ein öffentlicher Einstiegspunkt (Reverse Proxy)
- Alles andere ist intern
- Keine unnötigen offenen Ports

**Konsequenz:** Weniger Wege für Angreifer

### Interne Kommunikation

Kommunikation zwischen internen Services erfolgt im Docker-Netzwerk:
- Nicht übers Internet
- Nicht verschlüsselt nötig (da intern)
- Nur für berechtigte Services sichtbar

**Konsequenz:** Sichere interne Kommunikation

### Netzwerksegmentierung

Das Netzwerk ist segmentiert nach Verantwortlichkeit:
- `lld`-Netzwerk: Kern-Services
- Future: Separate Netzwerke für Monitoring, Kunden, etc.

Services in verschiedenen Segmenten können nicht direkt kommunizieren.

**Konsequenz:** Isolation von Services

### Kein unnötiger öffentlicher Zugriff

Datenbankserver, Admin-Interfaces und interne Services haben **keine** öffentliche Schnittstelle:
- Nicht im Internet erreichbar
- Nicht mappt auf öffentliche Ports
- Nicht ohne VPN/SSH-Tunnel zugänglich

**Konsequenz:** Services sind geschützt

## Verschlüsselung

### TLS für externe Kommunikation

Alle externe Kommunikation erfolgt über TLS/SSL (HTTPS):
- Vom Client zum Reverse Proxy: Verschlüsselt
- HTTP wird zu HTTPS weitergeleitet
- Zertifikate sind gültig und aktuell

**Konsequenz:** Daten in Transit sind geschützt

### Authentifizierung und Autorisierung

Jeder Service stellt sicher, dass nur autorisierte Requests verarbeitet werden:
- Authentifizierung: Wer sind Sie?
- Autorisierung: Darf dieser Nutzer das?

Dies ist Verantwortung der Anwendungen.

**Konsequenz:** Unbefugte können nicht auf Ressourcen zugreifen

## Logging und Auditing

### Protokollierung sicherheitsrelevanter Ereignisse

Folgende Ereignisse werden protokolliert:
- **Reverse Proxy:** Welche Requests kommen an? Von wo? Mit welchem Ergebnis?
- **Anwendungen:** Login-Versuche, Fehler, kritische Operationen
- **Datenbank:** Verbindungen, Queries (optional, abhängig von Logging-Level)
- **System:** Fehler, Warnungen, kritische Meldungen

**Nicht geloggt:** Sensitive Daten (Passwörter, Keys) in Logs

**Konsequenz:** Ereignisse sind nachvollziehbar

### Nachvollziehbarkeit

Logs ermöglichen es, zu rekonstruieren:
- Was ist passiert?
- Wann ist es passiert?
- Wer/Was hat es ausgelöst?
- Welches Ergebnis?

Dies ist essentiell für Debugging und Forensik.

### Dokumentation sicherheitsrelevanter Änderungen

Alle Änderungen an Sicherheitsaspekten werden dokumentiert:
- Neue Secrets werden rotiert
- Security-Patches werden eingespielt
- Zugriffskontrolle wird angepasst
- Zertifikate werden erneuert

**Dokumentation:** CHANGELOG.md, Commit-Messages, ADRs

**Konsequenz:** Änderungen sind nachverfolgbar

## Nicht erlaubt

Die folgenden Praktiken sind **explizit verboten**:

### Root-Container ohne Begründung

Container sollten **nicht als Root laufen**, außer es gibt einen zwingenden Grund.

Die meisten Services können mit eingeschränkten Rechten laufen.

**Grund:** Root-Container können das ganze System kompromittieren

**Ausnahme:** Services, die systemnahe Operationen brauchen (z.B. SSL-Zertifikate)

### Öffentlich erreichbare Datenbanken

Datenbanken sind **nicht von außen erreichbar**:
- ❌ Port 5432 ist nicht vom Internet offen
- ✅ Port 5432 ist nur intern im Docker-Netzwerk

**Grund:** Datenbanken sind kritische Systeme und können nicht selbst authentifizieren

**Konsequenz:** Nur authorized Services können auf Datenbanken zugreifen

### Hardcodierte Zugangsdaten

Keine Passwörter, API-Keys oder Secrets im Code:
- ❌ `password = "secret123"` im Source-Code
- ✅ `password = os.getenv("DATABASE_PASSWORD")`

**Grund:** Code kann geleakt werden, Secrets wären damit kompromittiert

**Siehe:** [docs/standards/environments.md](environments.md)

### Unverschlüsselte Kommunikation

Externe Kommunikation ist **immer** verschlüsselt:
- ❌ HTTP ohne HTTPS
- ✅ Automatische HTTP → HTTPS Redirection

**Grund:** Unverschlüsselte Daten können abgefangen werden

### Umgehung definierter Standards

Sicherheitsstandards werden **nicht umgangen**:
- ❌ SSH in Production-Container statt Deployment
- ❌ Manuelle Hotfixes statt Git-Workflow
- ❌ Direkter Datenbankzugriff statt über Anwendung

**Grund:** Umgehungen führen zu unkontrollierten Zuständen

**Konsequenz:** Standards werden konsequent durchgesetzt

### Unzureichend konfigurierte Healthchecks

Produktive Services haben **Healthchecks**:
- Sie prüfen regelmäßig ob Service läuft
- Bei Fehler werden Services neu gestartet
- Monitoring wird benachrichtigt

Services ohne Healthchecks können "still" sterben.

### Fehlende Aktualisierungen

Security-Patches und Updates werden **nicht ignoriert**:
- Images werden regelmäßig aktualisiert
- Abhängigkeiten werden auf Sicherheitslücken gepfüft
- Known Vulnerabilities werden gefixt

## Vulnerability Management

### Security Scanning

Regelmäßig werden Scanner verwendet, um Sicherheitslücken zu finden:
- Image-Scanning (CVEs in Dependencies)
- Dependency-Scanning
- Code-Scanning

Gefundene Lücken werden priorisiert und gefixt.

### Incident Response

Im Schadensfall:
- Incident wird dokumentiert
- System wird isoliert (falls nötig)
- Ursache wird analysiert
- Fix wird eingespielt
- Post-Mortem wird durchgeführt

## Langfristige Entwicklung

### Security-Scanning-Automatisierung

In späteren Phasen werden Security-Scans automatisiert:
- CI/CD-Pipeline lädt Images ein und scanned auf CVEs
- Abhängigkeiten werden gescanned
- Berichte werden generiert
- Sichere Deployments werden erzwungen

### Container-Hardening

Weitere Härtung von Containern:
- Immutable Filesystems
- Seccomp-Profiles
- AppArmor / SELinux
- Resource Limits (CPU, Memory)

### Intrusion Detection

Zukünftig können Intrusion-Detection-Systeme eingebaut werden:
- Anomaly Detection
- Network Monitoring
- Behavioral Analysis

### Automatisierte Sicherheitsprüfungen

Vor Production-Deployments:
- Automatische Security-Checks
- Policy-Enforcement
- Compliance-Validation

### Zero-Trust Architecture

Erweiterte Sicherheit:
- Keine Annahmen über interne Netzwerk-Sicherheit
- Alles wird verifiziert
- Minimale Zugriffspfade

## Zusammenfassung

Die Security-Standards der LLD-Infrastruktur sind:

- **Prinzipienbasiert** — Basiert auf bewährten Sicherheitsprinzipien
- **Defense in Depth** — Mehrere Sicherheitsschichten
- **Secure by Default** — Sicherheit ist Standard, nicht Optional
- **Nachvollziehbar** — Sicherheitsrelevante Ereignisse werden protokolliert
- **Zukunftsorientiert** — Erweiterbar für höhere Sicherheitsanforderungen

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Security Checklist

Vor jedem Deployment überprüfen:

- ✅ Keine Secrets im Code oder Git?
- ✅ TLS/HTTPS ist aktiviert?
- ✅ Datenbankzugriff ist beschränkt?
- ✅ Reverse Proxy ist der einzige öffentliche Entry-Point?
- ✅ Interne Services sind nicht öffentlich erreichbar?
- ✅ Healthchecks sind konfiguriert?
- ✅ Logging ist aktiviert?
- ✅ Alle Standards sind eingehalten?
- ✅ Security-Audit wurde durchgeführt?

---

## Verwandte Dokumente
- [docs/standards/environments.md](environments.md) — Environment und Secret Standards
- [docs/architecture/networking.md](../architecture/networking.md) — Netzwerk-Architektur
- [docs/architecture/reverse-proxy.md](../architecture/reverse-proxy.md) — Reverse Proxy
- [docs/standards/docker.md](docker.md) — Docker-Standards
