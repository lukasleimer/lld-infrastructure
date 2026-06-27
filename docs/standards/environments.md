# Environment und Konfiguration Standards — lld-infrastructure

Verbindliche Standards für die Verwaltung von Konfiguration und Umgebungsvariablen in der LLD-Infrastruktur.

## Ziel des Environment-Standards

Die Environment-Standards definieren die verbindlichen Regeln für den Umgang mit Konfiguration, Umgebungsvariablen und Secrets.

Ziele:

### Trennung von Code und Konfiguration

Code ist statisch und wird versioniert. Konfiguration ist dynamisch und umgebungsspezifisch.

Diese strikte Trennung ermöglicht es, dass der gleiche Code-Deployment in verschiedenen Umgebungen (Development, Staging, Production) läuft.

### Sicherheit

Secrets (Passwörter, API-Keys) werden **niemals** im Code oder Git gespeichert.

Dies ist eine kritische Sicherheitsanforderung.

### Wartbarkeit

Alle Konfigurationen sind dokumentiert und zentral verwaltbar.

Neue Team-Mitglieder können schnell verstehen, welche Variablen erforderlich sind.

### Portabilität

Die gleiche Code-Basis läuft in verschiedenen Umgebungen mit verschiedener Konfiguration.

Keine Hardcodes für spezifische Umgebungen.

### Reproduzierbarkeit

Deployments sind reproduzierbar, weil Konfiguration explizit und versioniert ist.

## Grundprinzipien

### Konfiguration erfolgt ausschließlich über Umgebungsvariablen

Jede konfigurierbare Einstellung wird als Umgebungsvariable definiert:
- Datenbankverbindungen
- API-Endpoints
- Debug-Modi
- Feature-Flags
- Timeouts und Limits

**Nicht:** Hardcodierte Werte im Code oder in Dateien

**Konsequenz:** Konfiguration ist flexibel und umgebungsspezifisch

### Secrets werden niemals im Quellcode gespeichert

Passwörter, API-Keys, Token und Zertifikate werden **niemals** im Git-Repository oder Quellcode gespeichert.

Sie werden:
- Nur als Umgebungsvariablen bereitgestellt
- In sicheren Secret-Management-Systemen gespeichert
- Mit strenger Zugriffskontrolle verwaltet

**Grund:** Ein Commit mit einem Secret ist ein Sicherheitsvorfall

**Konsequenz:** Code-Repository kann öffentlich sein, ohne Sicherheitsrisiko

### `.env.example` dokumentiert alle erforderlichen Variablen

Eine `.env.example`-Datei liegt im Repository und dokumentiert **alle erforderlichen Umgebungsvariablen**:
- Variablennamen
- Datentyp (String, Integer, Boolean)
- Zweck und Beschreibung
- Optionale Standardwerte (für nicht-sensible Werte)

`.env.example` wird versioniert. `.env` wird **nicht** versioniert.

**Konsequenz:** Neue Developer wissen sofort, welche Variablen erforderlich sind

### Jede Variable wird dokumentiert

Jede Umgebungsvariable hat einen eindeutigen Namen und eine Beschreibung:

**Dokumentation beinhaltet:**
- **Name:** Eindeutige Variable
- **Typ:** String, Integer, Boolean, URL, etc.
- **Zweck:** Wozu wird diese Variable verwendet?
- **Beispielwert:** Ein repräsentativer (nicht-sensitiver) Beispielwert
- **Erforderlich:** Ja/Nein
- **Standard:** Optionaler Standardwert

**Konsequenz:** Wartbarkeit und Verständnis ist hoch

### Hardcodierte Konfiguration ist nicht zulässig

Alle Konfiguration muss dynamisch aus Umgebungsvariablen kommen.

Hardcodes sind **nicht erlaubt**:
- ❌ `DATABASE_URL = "postgres://localhost:5432/db"`
- ✅ `DATABASE_URL = os.getenv("DATABASE_URL")`

**Ausnahme:** Dokumentation, nicht-sensitive Beispielwerte

**Konsequenz:** Code ist konfigurierbar und portabel

## Environment-Dateien

### `.env.example` — Template für Entwickler

Die `.env.example`-Datei liegt im Repository und dient als Template.

Sie enthält:
- Alle erforderlichen Variablennamen
- Beschreibungen
- Nicht-sensitive Beispielwerte
- Kommentare und Dokumentation

Sie enthält **nicht:**
- Echte Passwörter oder Secrets
- Production-Werte
- Sensitive Daten

**Status:** Versioniert in Git

**Verwendung:** Entwickler kopieren diese zu `.env` und füllen echte Werte ein

### `.env` (lokal) — Entwickler-Konfiguration

Die `.env`-Datei ist die **lokale Umgebungskonfiguration** für jeden Entwickler.

Sie liegt **nicht** im Repository (`.gitignore`).

Sie enthält:
- Kopie von `.env.example`
- Mit echten, lokalen Werten gefüllt
- Lokale Passwörter und Secrets

**Status:** Nicht versioniert (in `.gitignore`)

**Verwendung:** Docker und Anwendungen lesen diese lokal

### Produktive Umgebungsvariablen

In Production werden Umgebungsvariablen durch ein Secret-Management-System bereitgestellt:
- Environment-Variablen direkt im Container
- Secret-Store (z.B. Vault, AWS Secrets Manager)
- Sichere Konfigurationsmanagement-Systeme

**Status:** Zentral verwaltet und gesichert

**Verwendung:** Container erhält diese beim Start

### Validierung fehlender Variablen

Die Anwendung **validiert** beim Start, dass alle erforderlichen Variablen gesetzt sind.

**Ausnahme:** Variablen mit Standardwerten sind optional

**Fehlerfall:** 
- Klare Fehlermeldung welche Variable fehlt
- Container stoppt nicht unerwartet später, sondern early

**Konsequenz:** Deployment-Fehler werden früh erkannt

## Secrets — Sichere Konfiguration

### Passwörter und Datenbank-Zugangsdaten

Passwörter für Datenbankzugriff werden als Umgebungsvariablen bereitgestellt:
- `DATABASE_PASSWORD` oder Teil von `DATABASE_URL`
- Niemals hardcoded oder versioniert
- Nur in Secret-Storage

**Regel:** Passwörter sind mindestens 16 Zeichen stark

### API-Keys und Tokens

Externe API-Keys (z.B. für Services, Gateways) werden als Umgebungsvariablen:
- `EXTERNAL_API_KEY`
- `JWT_SECRET`
- `STRIPE_API_KEY`

Niemals in Code oder Git.

### Zertifikate

SSL-Zertifikate und private Keys werden:
- In Umgebungsvariablen oder als Dateien in Volumes
- Mit strenger Zugriffskontrolle
- Niemals im Repository

### Secrets Manager

In späteren Phasen wird ein Secrets Manager verwendet:
- Zentrale Verwaltung aller Secrets
- Versioning und Auditing
- Automatische Rotation
- Zugriffskontrolle

**Konsequenz:** Secrets sind zentral verwaltet und sicher

## Namenskonventionen für Umgebungsvariablen

### Format

**Format:** `{PROJECT}_{COMPONENT}_{PROPERTY}` oder `{COMPONENT}_{PROPERTY}`

**Beispiele:**
- `LLD_DATABASE_URL` — Datenbankverbindung
- `LLD_PANEL_DEBUG` — Debug-Modus für Panel
- `PANEL_SECRET_KEY` — Secret Key der Panel-Anwendung
- `POSTGRES_PASSWORD` — PostgreSQL-Passwort

### Regeln

- **GROSSBUCHSTABEN**
- Nur Buchstaben, Ziffern und Unterstriche (`_`)
- Aussagekräftig und selbsterklärend
- Nicht: `X`, `VAR1`, `CONFIG`

### Standardisierte Variablen

Häufig verwendete Variablen haben Standard-Namen:
- `DATABASE_URL` — Datenbankverbindung (nicht `DB_CONNECTION`)
- `DEBUG` oder `DEBUG_MODE` — Debug-Flag
- `LOG_LEVEL` — Logging-Level
- `ENVIRONMENT` oder `ENV` — Environment Name (development/staging/production)

**Grund:** Konsistenz und Klarheit

## Nicht erlaubt

Die folgenden Praktiken sind **explizit verboten**:

### Hardcodierte Zugangsdaten

Keine Passwörter oder Secrets im Source-Code:
- ❌ `DATABASE_PASSWORD = "super_secret_123"`
- ❌ Hardcodierte API-Keys
- ❌ Secrets in Konfigurationsdateien

**Grund:** Security-Risiko, Code kann nicht öffentlich sein

**Stattdessen:** Umgebungsvariablen
- ✅ `DATABASE_PASSWORD = os.getenv("DATABASE_PASSWORD")`

### Secrets im Git-Repository

Secrets dürfen **niemals** committed werden:
- ❌ `.env`-Datei mit echten Werten
- ❌ Credentials-Dateien
- ❌ Private Keys

**Grund:** Ein Commit ist für immer in Git-Historien

**Stattdessen:**
- `.env.example` als Template (ohne Secrets)
- `.env` in `.gitignore`

### Inkonsistente Variablennamen

Verschiedene Namen für die gleiche Konfiguration sind **nicht erlaubt**:
- ❌ `DATABASE_URL` in einer Komponente, `DB_CONNECTION` in anderer
- ❌ `PANEL_DEBUG` und `DEBUG_PANEL` (gemischt)

**Stattdessen:** Konsistente, standardisierte Namen
- ✅ `DATABASE_URL` überall gleich

### Ungenutzte oder undokumentierte Variablen

Jede Umgebungsvariable muss:
- Verwendet werden (nicht "für später")
- Dokumentiert sein (Zweck klar)
- In `.env.example` stehen

Verwaiste oder geheime Variablen sind **nicht erlaubt**.

### Variablen in mehreren Quellen

Konfiguration aus verschiedenen Quellen (Code, Dateien, Umgebung) ist **nicht erlaubt**.

**Stattdessen:** Eine einzige Quelle (Umgebungsvariablen)

### Sensitives in Logs

Umgebungsvariablen mit Secrets dürfen **nicht** in Logs erscheinen:
- ❌ `"Connected with password: secret123"`
- ✅ `"Database connected"` (ohne Details)

**Konsequenz:** Logs können öffentlich gemacht werden

## Umgebungen

### Development

Lokal auf Entwickler-Maschinen:
- `.env` mit lokalen Werten
- Nicht-sensitive Standard-Werte
- Debug-Modus aktiv (optional)
- Einfache Secrets möglich

### Staging

Test-Umgebung (ähnlich Production):
- Konfiguration via Secret-Store
- Production-ähnliche Werte
- Real-ähnliche Daten (anonymisiert)
- Monitoring aktiv

### Production

Live-Umgebung:
- Konfiguration via Secret-Store
- Real-Werte
- Maximale Sicherheit
- Audit-Logging

**Umgebungs-Identifier in Variablen:**
- `ENVIRONMENT=production`
- Oder separat verwaltete Dateien/Stores

## Konfiguration validieren und dokumentieren

### Validierung beim Start

Die Anwendung prüft beim Start:
- Alle erforderlichen Variablen sind gesetzt
- Werte sind vom richtigen Typ
- Werte sind im zulässigen Bereich

**Fehlerfall:** Aussagekräftige Fehlermeldung
- Welche Variable fehlt?
- Was ist erforderlich?
- Wie wird es gesetzt?

### `.env.example` als Dokumentation

`.env.example` dient als primäre Dokumentation:

```
# Database Configuration
DATABASE_URL=postgres://user:password@localhost:5432/lld_panel
DATABASE_POOL_SIZE=5

# Panel Configuration
PANEL_DEBUG=false
PANEL_SECRET_KEY=your-secret-key-here

# Logging
LOG_LEVEL=info
```

Jede Variable ist kommentiert und erklärt.

## Audit und Sicherheit

### Secret-Rotation

Secrets sollten regelmäßig rotiert werden:
- Passwörter alle 3-6 Monate
- API-Keys bei Verdacht auf Kompromittierung
- Zertifikate vor Ablauf erneuern

### Zugriffskontrolle

Nur autorisierte Personen haben Zugriff auf Production-Secrets:
- Entwickler: Development-Secrets
- DevOps/SRE: Production-Secrets
- Dokumentation: Wer hat Zugriff?

### Audit-Logging

Alle Zugriffe auf Secrets werden protokolliert:
- Wer hat zugegriffen?
- Wann?
- Welche Variablen?

## Langfristige Entwicklung

### Secret-Management-System

Ein dediziertes System zur Secret-Verwaltung:
- Zentrale Verwaltung
- Automatische Rotation
- Versioning
- Zugriffskontrolle
- Audit-Logging

**Beispiele:** Hashicorp Vault, AWS Secrets Manager

**Konsequenz:** Professionelle Secret-Verwaltung

### Verschlüsselte Konfiguration

Konfigurationsdateien können verschlüsselt werden:
- At-Rest-Verschlüsselung
- Schlüsselverwaltung
- Sichere Übertragung

### Automatisierte Validierung

In CI/CD können Environment-Standards überprüft werden:
- Keine Secrets in Code?
- Alle Variablen dokumentiert?
- Naming-Standards eingehalten?

**Konsequenz:** Sicherheit wird erzwungen

### Integration mit LLD Panel

Das Admin-Panel kann Umgebungsvariablen verwalten:
- UI zum Setzen von Variablen
- Sichere Speicherung
- Audit-Logging
- Automatische Validation

**Konsequenz:** Einfache, sichere Konfigurationsverwaltung

### Multi-Environment-Management

Bei vielen Umgebungen (Development, Staging, Production):
- Pro Environment: Separate Secrets
- Easy Switching zwischen Environments
- Isolation und Sicherheit pro Environment

## Zusammenfassung

Die Environment-Standards der LLD-Infrastruktur sind:

- **Sicher** — Keine Secrets im Code
- **Wartbar** — Alle Variablen dokumentiert
- **Portabel** — Code läuft in verschiedenen Umgebungen
- **Konsistent** — Standardisierte Variablennamen
- **Nachvollziehbar** — Audit und Logging

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Checkliste für neue Umgebungsvariablen

Bevor eine neue Umgebungsvariable verwendet wird:

- ✅ Namen folgt Namenskonventionen (`{PROJECT}_{COMPONENT}_{PROPERTY}`)
- ✅ Variable ist in `.env.example` dokumentiert
- ✅ Dokumentation enthält Zweck und Beispielwert
- ✅ Keine Secrets werden hardcoded
- ✅ Variable wird validiert beim Start (falls erforderlich)
- ✅ Fehlermeldungen sind aussagekräftig
- ✅ Variable wird nicht in Logs geloggt (falls sensitiv)
- ✅ Dokumentation ist aktuell

---

## Verwandte Dokumente
- [docs/standards/docker.md](docker.md) — Docker-Standards
- [docs/standards/naming.md](naming.md) — Naming-Standards
- [docs/architecture/overview.md](../architecture/overview.md) — Allgemeine Architekturübersicht
