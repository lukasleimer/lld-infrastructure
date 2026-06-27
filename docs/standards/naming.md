# Naming-Standards — lld-infrastructure

Verbindliche Namenskonventionen für alle Ressourcen innerhalb der LLD-Infrastruktur.

## Ziel des Naming-Standards

Namenskonventionen definieren die verbindlichen Regeln für die Benennung aller Ressourcen der Plattform.

Ziele:

### Einheitlichkeit

Alle Ressourcen folgen denselben Benennungsmustern. Dies ermöglicht schnelles Verständnis und Konsistenz.

### Lesbarkeit

Namen sind selbstsprechend und beschreiben die Funktion oder den Zweck der Ressource.

### Vorhersehbarkeit

Aus dem Namen kann man den Typ und Zweck einer Ressource erraten.

Beispiel: `customer-001-postgres` macht deutlich, dass es eine Postgres-Instanz für Kunde 001 ist.

### Wartbarkeit

Konsistente Namen machen die Plattform wartbar und Fehler vermeidbar.

## Allgemeine Regeln

### Kleinschreibung

Alle Namen verwenden **ausschließlich Kleinbuchstaben**:
- ✅ `lld-panel`
- ✅ `customer-001`
- ❌ `LLD-Panel`
- ❌ `Customer-001`

**Grund:** Konsistenz, Portabilität (manche Systeme sind case-sensitive)

### Keine Leerzeichen

Namen enthalten **keine Leerzeichen**:
- ✅ `lld-postgres-data`
- ❌ `lld postgres data`

**Grund:** Leerzeichen verursachen Parsing-Probleme in vielen Systemen

### Keine Sonderzeichen (außer `-` und `_`)

Names verwenden **nur** Buchstaben, Ziffern, Bindestriche (`-`) und Unterstriche (`_`):
- ✅ `customer-001`
- ✅ `customer_001`
- ❌ `customer#001`
- ❌ `customer@001`
- ❌ `customer.001`

**Grund:** Nur diese Zeichen sind in allen Systemen sicher

### Aussagekräftige Namen

Namen beschreiben **Funktion oder Zweck**:
- ✅ `lld-postgres` (klar: Datenbankserver der LLD-Plattform)
- ✅ `lld-uploads` (klar: Upload-Verzeichnis)
- ❌ `db1`, `vol1` (generisch und unklar)

**Grund:** Aussagekräftige Namen ermöglichen schnelle Orientierung

### Bindestriche für Services/Container

Services, Container und externe Identifizierer verwenden **Bindestriche** (`-`):
- `lld-panel`
- `lld-website`
- `customer-001`
- `reverse-proxy`

### Unterstriche für Volumes/Datenbanken

Volumes und Datenbanken verwenden **Unterstriche** (`_`):
- `lld_postgres_data`
- `lld_uploads`
- `customer_001_data`

**Grund:** Verschiedene Systeme haben verschiedene Präferenzen; Konvention schafft Klarheit

## Namenskonventionen

### Container und Services

**Format:** `{project}-{component}`

**Beispiele:**
- `lld-panel` — Admin-Panel-Container
- `lld-website` — Unternehmenswebseite-Container
- `lld-postgres` — Datenbankserver-Container
- `reverse-proxy` — Reverse Proxy Container

**Kundenservices:**

**Format:** `customer-{id}-{component}`

**Beispiele:**
- `customer-001-website` — Kundenwebseite für Kunde 001
- `customer-002-website` — Kundenwebseite für Kunde 002

**Regel:** Service-Namen verwenden **Bindestriche** und **Kleinschreibung**

### Docker-Netzwerke

**Format:** `{project}-network` oder einfach `{project}`

**Beispiele:**
- `lld` — Hauptnetzwerk für LLD-Services
- `lld-monitoring` — Netzwerk für Monitoring-Services (zukünftig)

**Regel:** Netzwerke verwenden **Bindestriche** und sind **aussagekräftig**

### Docker Volumes

**Format:** `{project}_{purpose}` oder `{customer}_{purpose}`

**Beispiele:**
- `lld_postgres_data` — PostgreSQL-Datenbank-Dateien
- `lld_uploads` — Benutzerdateien
- `lld_config` — Konfigurationsdateien
- `customer_001_data` — Daten für Kundenwebseite 001
- `customer_001_uploads` — Uploads für Kundenwebseite 001

**Varianten:**
- `{project}-{purpose}-data` — Alternative mit Bindestrichen, wenn Unterstriche nicht unterstützt

**Regel:** Volumes verwenden **Unterstriche** zur Trennung und enden ggf. mit `_data`

### PostgreSQL-Datenbanken

**Format:** `{project}_{component}` oder `customer_{id}`

**Beispiele:**
- `lld_panel` — Datenbank für Panel-Anwendung
- `lld_website` — Datenbank für Website-Anwendung
- `customer_001` — Datenbank für Kundenwebseite 001

**Benutzernamen:**

**Format:** `{project}_{app}_user` oder `customer_{id}_user`

**Beispiele:**
- `lld_panel_user` — Benutzer für Panel-Datenbank
- `customer_001_user` — Benutzer für Kundendatenbank

**Regel:** Datenbanken verwenden **Unterstriche** zur Trennung

### PostgreSQL-Tabellen

**Format:** `{entity_plural}` oder `{namespace}_{entity_plural}`

**Beispiele:**
- `users` — Benutzertabelle
- `uploads` — Upload-Tabelle
- `customer_settings` — Einstellungen für Kunden

**Regel:** Tabellennamen verwenden **Unterstriche**, Plural-Form, **Kleinschreibung**

### Domains und URLs

**Format:** `{component}.{base_domain}`

**Beispiele:**
- `panel.lukasleimer.dev` — Admin-Panel
- `lukasleimer.dev` — Unternehmenswebseite
- `kunde-1.kundendomains.de` — Kundenwebseite (eigene Domain)

**Subdomains für Kunden:**
- `customer-001.panel.lukasleimer.dev` — Admin-Interface für Kunde 001 (zukünftig)

**Regel:** Domains verwenden **Kleinschreibung**, **Bindestriche** bei Subdomains

### Git-Repositories

**Format:** `{project}-{component}` oder `lld-{component}`

**Beispiele:**
- `lld-infrastructure` — Dieses Repository (Infrastruktur)
- `lld-panel` — Admin-Panel-Quellcode
- `lld-website` — Website-Quellcode
- `lld-customer-template` — Template für Kundenwebseiten

**Branch-Namen:**

**Format:** `{type}/{feature}` oder `{type}/#{issue}`

**Beispiele:**
- `feature/docker-compose` — Neues Feature
- `fix/networking-issue` — Fehlerbehebung
- `docs/architecture` — Dokumentation
- `hotfix/security-patch` — Kritischer Fix

**Regel:** Branches verwenden **Kleinschreibung**, **Bindestriche**

### Kunden-Identifizierer

**Format:** `customer-{id}` oder `customer_{id}` (abhängig vom Kontext)

**Beispiele:**
- `customer-001` — Container/Service
- `customer_001` — Datenbank/Volume
- `customer-001-website` — Webseite
- `customer_001_data` — Daten-Volume

**ID-Format:** 
- Ziffern mit führenden Nullen: `001`, `002`, ... `999`
- Für Domains: Identifier möglich: `kunde-at`

**Regel:** Kundennummern sind **dreistellig mit führenden Nullen** für Sortierbarkeit

### Umgebungsvariablen

**Format:** `{PROJECT}_{COMPONENT}_{PROPERTY}`

**Beispiele:**
- `LLD_PANEL_DEBUG` — Debug-Modus für Panel
- `LLD_DATABASE_URL` — Datenbankverbindung
- `LLD_UPLOADS_PATH` — Pfad zu Uploads

**Regel:** Umgebungsvariablen verwenden **GROSSBUCHSTABEN**, **Unterstriche** zur Trennung

### Monitoring und Logging

**Metriken-Namen:**

**Format:** `{project}_{component}_{metric}`

**Beispiele:**
- `lld_panel_requests_total` — Gesamtanzahl Requests
- `lld_postgres_connections_active` — Aktive Datenbankverbindungen

**Log-Labels:**

**Format:** `component=value, service=value`

**Beispiele:**
- `component=lld-panel, service=web`
- `component=customer-001-website, customer=001`

**Regel:** Metriken verwenden **Unterstriche**, Labels verwenden **Bindestriche** in Werten

## Präfixe

### `lld-` (Bindestrich-Präfix)

Wird verwendet für:
- **Container und Services** der LLD-Plattform
- **Externe Identifizierer** (Domains)
- **Branch-Namen**
- **Repository-Namen**

**Beispiele:**
- `lld-panel` (Container)
- `lld-website` (Container)
- `panel.lukasleimer.dev` (Domain)
- `lld-infrastructure` (Repository)

### `lld_` (Unterstrich-Präfix)

Wird verwendet für:
- **Volumes** und Dateisystem-Ressourcen
- **Datenbanken**
- **Tabellen und Datenbank-Objekte**
- **Umgebungsvariablen**

**Beispiele:**
- `lld_postgres_data` (Volume)
- `lld_panel` (Datenbank)
- `lld_users` (Tabelle)
- `LLD_DATABASE_URL` (Umgebungsvariable)

### `customer-` (Bindestrich-Präfix)

Wird verwendet für:
- **Customer-Container und Services**
- **Customer-Domains**
- **Repository-Namen für Kunden-Template**

**Beispiele:**
- `customer-001-website` (Container)
- `customer-1.kundendomains.de` (Domain)

### `customer_` (Unterstrich-Präfix)

Wird verwendet für:
- **Customer-Volumes**
- **Customer-Datenbanken**
- **Customer-spezifische Konfigurationen**

**Beispiele:**
- `customer_001_data` (Volume)
- `customer_001` (Datenbank)

## Nicht erlaubt

Die folgenden Namensmuster sind **explizit verboten**:

### Generische Namen

Generische, nicht aussagekräftige Namen sind **nicht erlaubt**:
- ❌ `test`, `app`, `db`, `service`, `container`
- ❌ `vol1`, `vol2`, `db1`, `db2`
- ❌ `config`, `data` (zu vage)

**Stattdessen:** Aussagekräftige Namen mit Präfixen
- ✅ `lld-panel`
- ✅ `lld_uploads`
- ✅ `customer_001_data`

### Automatisch generierte Namen

Automatisch generierte UUIDs oder Hashes als Namen sind **nicht erlaubt**:
- ❌ `abc123def456`
- ❌ `volume_a8f9e3c1`

**Stattdessen:** Explizit benannte Ressourcen
- ✅ `lld_postgres_data`

### Inkonsistente Schreibweisen

Verschiedene Schreibweisen der gleichen Ressource sind **nicht erlaubt**:
- ❌ `LLD-Panel`, `lld-panel`, `Lld-Panel` (gemischt)
- ❌ `lld-panel`, `lldPanel` (Camel-Case)

**Stattdessen:** Konsistente Kleinschreibung
- ✅ `lld-panel` (überall gleich)

### Sonderzeichen außer `-` und `_`

Punkte, Underscores in falschen Kontexten oder andere Zeichen sind **nicht erlaubt**:
- ❌ `lld.panel` (Punkt statt Bindestrich)
- ❌ `lld-panel_data` (gemischte Präfixe)
- ❌ `lld#panel` (ungültiges Zeichen)

**Stattdessen:** Nur `-` und `_` verwenden
- ✅ `lld-panel` (Container/Service)
- ✅ `lld_panel` (Datenbank)

### Schlechte Abstraktionen

Namen, die technische Details statt Zweck beschreiben, sind **nicht erlaubt**:
- ❌ `docker-compose-v1`, `yaml-config`
- ❌ `data-layer-2`

**Stattdessen:** Funktionsbeschreibende Namen
- ✅ `lld_uploads`
- ✅ `lld_postgres_data`

## Spezialfälle

### Temporäre Ressourcen

Temporäre oder Test-Ressourcen können mit Präfix gekennzeichnet werden:
- `test-lld-panel` — Test-Instanz des Panels
- `dev-customer-001` — Development-Instanz eines Kunden

**Regel:** Prefix vor dem Standardnamen, nicht nachher

### Versionen

Wenn mehrere Versionen existieren, kann eine Nummer angehängt werden:
- `lld_postgres_data_v1` — Datenbank-Backup Version 1
- `lld_uploads_backup_20260601` — Datum-basiertes Backup

**Regel:** Versionen am Ende, durch Unterstrich getrennt

### Namenskollisionen

Falls Namenskollisionen auftreten, wird die Service-Ebene präzisiert:
- `lld-panel-production`
- `lld-panel-staging`
- `lld-panel-development`

**Regel:** Environment am Ende hinzufügen

## Governance

### Naming Review

Neue Ressourcen sollten im Code-Review überprüft werden:
- Folgen sie den Standards?
- Sind sie aussagekräftig?
- Gibt es Kollisionen mit bestehenden Namen?

### Umbenennungen

Wenn eine Ressource umbenannt wird:
- Die alte Konfiguration wird entfernt
- Die neue wird dokumentiert
- Das Changelog wird aktualisiert

## Langfristige Entwicklung

### Automatisierte Namensvalidierung

In späteren Phasen kann eine automatisierte Validierung eingeführt werden:
- Git-Hooks prüfen Naming-Standards
- Linting in CI/CD
- Automatische Rejection von falschen Namen

**Konsequenz:** Standards werden erzwungen.

### Multi-Customer-Hosting

Mit vielen Kunden wird ein Naming-System notwendig:
- Customer-IDs eindeutig (001, 002, ... 999)
- Pro Kunde: Volumes, Datenbanken, Services mit eindeutigem Namen
- Keine Namenskollisionen möglich

**Konsequenz:** Skalierbarkeit auf beliebig viele Kunden.

### Panel-Integration

Das LLD Panel kann Namen automatisch generieren:
- Neue Kundenwebseite → automatische Namen für alle Ressourcen
- Keine manuellen Fehler mehr
- Konsistenz garantiert

**Konsequenz:** Fehlerfreie und konsistente Naming.

### Automatisierte Skripte

Skripte zum Auflisten, Verwalten und Monitoren können Namen verwenden:
- `show-volumes lld` — Alle LLD-Volumes
- `backup customer-001` — Backup für Kundenwebseite 001
- `logs lld-panel` — Logs für Panel

**Konsequenz:** Einfache Automation möglich.

## Zusammenfassung

Die Naming-Standards der LLD-Infrastruktur sind:

- **Konsistent** — Alle Ressourcen folgen gleichen Mustern
- **Aussagekräftig** — Namen beschreiben Funktion und Zweck
- **Vorhersehbar** — Man kann Namen erraten
- **Wartbar** — Standards machen Fehler unwahrscheinlich
- **Skalierbar** — Standards unterstützen Wachstum (Kunden, Services)

---

**Version:** 0.1.0-alpha.1  
**Letztes Update:** 2026-06-27

---

## Quick Reference

| Ressourcentyp | Format | Beispiel | Zeichen |
|---|---|---|---|
| Container/Service | `{project}-{component}` | `lld-panel` | `-` |
| Customer-Container | `customer-{id}-{component}` | `customer-001-website` | `-` |
| Docker Volume | `{project}_{purpose}` | `lld_postgres_data` | `_` |
| Customer-Volume | `customer_{id}_{purpose}` | `customer_001_data` | `_` |
| Datenbank | `{project}_{component}` | `lld_panel` | `_` |
| Customer-Datenbank | `customer_{id}` | `customer_001` | `_` |
| Domain | `{component}.{domain}` | `panel.lukasleimer.dev` | `-` |
| Git-Repository | `{project}-{component}` | `lld-infrastructure` | `-` |
| Git-Branch | `{type}/{feature}` | `feature/docker-compose` | `-`, `/` |
| Umgebungsvariable | `{PROJECT}_{COMPONENT}` | `LLD_DATABASE_URL` | `_`, GROSS |

---

## Verwandte Dokumente
- [docs/standards/docker.md](docker.md) — Docker-Standards
- [docs/standards/compose.md](compose.md) — Docker Compose Standards
- [docs/architecture/overview.md](../architecture/overview.md) — Allgemeine Architekturübersicht
