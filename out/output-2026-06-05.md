# df-ota-airbnb-adapter — Output [CRUX-MK]
*Autonom aktiviert 2026-06-05T15:47:51.725295+00:00 | ollama-local/qwen2.5:14b-instruct*

# df-ota-airbnb-adapter [CRUX-MK]

## Zweck

Der `df-ota-airbnb-adapter` ist ein Connector für die Airbnb Hosting-Platfo
Hosting-Platform-API, der folgende Funktionen umfasst:

- **Inventory-Query:** Abfrage der Verfügbarkeit von Listings.
- **Rate-Push:** Aktualisierung von Tarifen.
- **Booking-Pull + Webhook:** Pull-Buchungen und Webhooks (Benachrichtigung
(Benachrichtigungen) für neue Buchungen.
- **Kommissionen:** Ein 3%-Host-Kommissions-Tracker pro Booking.

## Vendor-API-Pattern

- **Hosting-Platform-API:** Listings, Bookings und Kalender abfragen.
- **OAuth-2.0-Authentifizierung:** Zugriff auf den Airbnb-Dienst über OAuth
OAuth-2.0-Token.
- **Webhook:** Benachrichtigungen mit HMAC-SHA256-Verifikation.
- **OAuth Access-Token + Refresh-Token:** Für die Authentifizierung und Akt
Aktualisierung der Tokens.

## Default-Modus: Sandbox

Der Adapter läuft standardmäßig im "Sandbox"-Modus (`DF_OTA_AIRBNB_REAL_ENA
(`DF_OTA_AIRBNB_REAL_ENABLED=false`). In diesem Modus werden Mock-Daten ver
verwendet. Um den Real-Modus zu aktivieren, müssen die folgenden Umgebungsv
Umgebungsvariablen gesetzt sein:

- `DF_OTA_AIRBNB_REAL_ENABLED=true`
- `AIRBNB_HOTELIER_ID`: Die ID des Hoteliers.
- `AIRBNB_API_KEY`: Der API-Schlüssel für den Zugriff auf Airbnb-Dienste.
- `DF_OTA_AIRBNB_PHRONESIS_TICKET`: Ein Ticket für Push-Vorgänge.

## Module

### Hauptmodule

- **src/airbnb_adapter.py:** Diese Datei enthält die Implementierung der Ho
Hosting-Platform-API und eine 3%-Kommissionen-Verfolgung.
- **src/airbnb_auth.py:** OAuth-2.0-Pattern, das auf Umgebungsvariablen bas
basiert.
- **src/airbnb_webhook.py:** Empfänger für Buchungsbenachrichtigungen mit H
HMAC-SHA256-Verifikation.

### Zusätzliche Module

- **commission_tracker.py:** Verfolgt pro-Buchungskommissionen und erstellt
erstellt Aggregatberichte.
- **adapter_orchestrator.py:** Entry Point des LaunchAgents.
- **audit_logger.py:** Protokolliert HMAC-SHA256-gesignierte Audit-Einträge
Audit-Einträge (JSONL append-only).

## Tests

Die Testdatei `tests/test_airbnb_adapter.py` enthält 27+ Tests, um den Adap
Adapter, die Authentifizierung, Webhooks, Tracker, Orchestrator und AuditLo
AuditLogger zu überprüfen. Um die Tests auszuführen:

```bash
cd df-ota-airbnb-adapter
PYTHONPATH=. python -m pytest tests/ -v
```

## Ausführung via LaunchAgent

Folgender Befehl kopiert den LaunchAgent-Pfad in das Home-Verzeichnis und l
lädt ihn dann ein:

```bash
cp scripts/com.kemmer.df-ota-airbnb-adapter.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.kemmer.df-ota-airbnb-adapter.plis
~/Library/LaunchAgents/com.kemmer.df-ota-airbnb-adapter.plist
```

Der LaunchAgent wird bei jedem Start ausgelöst (`RunAtLoad=true`) und läuft
läuft alle 2 Stunden (`StartInterval=7200`).

## K11-K16 Compliance

- **K11:** Cascade-Containment durch try/except + LC4 idempotent.
- **K12:** Provenance über frozen dataclass + source-tracking.
- **K13:** PAV (Privacy and Anonymity Verification) via env_tag + vendor_ap
vendor_api anchor.
- **K14:** Override via single_command + martin_review weekly.
- **K15:** Entropy ~700 LOC mit rho-Justifikation 40k EUR/Jahr.
- **K16:** Concurrent-Spawn-Mutex über mkdir-lock + pgrep.

## LC1-LC5 Compliance

- **LC1:** Drei Degradationsmodi (full / degraded_no_real_api / standalone_
standalone_mock).
- **LC2:** Direkte Modusfähigkeit 0.5 (Mock-Daten ohne Real-API).
- **LC3:** Circuit-Breaker (30s Timeout, 3 Fails, 300s Half-Open).
- **LC4:** Failure-Isolation durch state_externalization + idempotent_opera
idempotent_operations.
- **LC5:** Standalone-Healthcheck.