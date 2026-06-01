# CLAUDE.md

Diese Datei gibt Claude Code (claude.ai/code) Orientierung beim Arbeiten in diesem Repository.

## Was dieses Repo ist

Eine Sammlung von [Home Assistant Blueprints](https://www.home-assistant.io/docs/blueprint/). Jede `.yaml` ist eine eigenständige Blueprint, die direkt von der HA-Automationsmaschine verarbeitet wird — keine Build-Schritte, Tests oder Abhängigkeiten.

| Datei | Zweck |
|---|---|
| `Bodenfeuchte.yaml` | Bodenfeuchte-Alarmsystem (Trocken-Alarm, Zeitplan-Gießen, Pumpensteuerung, Multi-Channel-Benachrichtigung). |
| `alexa_timer_awtrix.yaml` | Zeigt den nächsten Alexa-Timer als Live-Countdown auf einem AWTRIX-3-Display und löst beim Ablauf einen Alarm aus. |

Installation jeweils in HA: **Einstellungen → Automationen & Szenen → Blueprints → Blueprint importieren**, mit der `source_url` am Ende des `blueprint:`-Abschnitts.

Die folgenden Abschnitte „Blueprint-Struktur" bis „Versionierung" beschreiben primär `Bodenfeuchte.yaml`; der Abschnitt ganz unten dokumentiert `alexa_timer_awtrix.yaml`.

## Blueprint-Struktur

| Abschnitt | Zweck |
|---|---|
| `blueprint.input` | Alle konfigurierbaren Felder, nach Kommentar-Überschriften gruppiert. Jedes Feld braucht `selector:` und meist ein `default:`. |
| `variables:` | Löst alle `!input`-Werte auf, normalisiert sie und berechnet abgeleitete Werte (`thr_ok`, `is_paused`, `person_in_zone`, `mobile_notify_services`). Alle Jinja2-Logik, die in mehreren Action-Blöcken gebraucht wird, gehört hierher. |
| `trigger:` | HA-Trigger-Definitionen (numeric state, time, event, …). |
| `action:` | Hauptsequenz aus `choose:`- und `sequence:`-Blöcken — je ein Block pro logischem Pfad (Trocken-Alarm, OK, Zeitplan-Gießen, Sensorausfall, Snooze/Dismiss-Events). |

`mode: restart` bedeutet: ein neuer Trigger startet die gesamte Automation neu. `max_exceeded: silent` unterdrückt die Warnung im Log.

## Input-Feld-Konventionen

- Logische Gruppen werden durch `# ─── Emoji ÜBERSCHRIFT`-Kommentare getrennt.
- Optionale Entity-/Device-Selektoren verwenden `default: null` — nicht `default: ""` oder `default: false` (beide verursachten HA-Validierungsfehler, siehe Changelog v1.9).
- Multi-Select-Entity-Felder nutzen `multiple: true` unter `selector.entity`.
- Guard-Muster vor jedem `states()`-Aufruf auf optionalen Entities:
  ```yaml
  {% if entity is not none and entity is not sameas false and (entity | string) != "None" %}
  ```
- Alexa-Targets werden in `variables:` via `alexa_targets_safe` zu einer sicheren Liste normalisiert, bevor irgendein `repeat: for_each:` darauf zugreift.

## Architektur-Muster

**Quiet-time-Puffer**: Wenn ein Kanal in seiner Ruhezeit ist, nutzt der Action-Block `wait_for_trigger` (nicht `delay:`) mit zwei Ausstiegsbedingungen — Ruhezeit-Ende oder Sensor-Erholung — gefolgt von einer `condition:`, die den Sensor nochmals prüft, bevor die Benachrichtigung gesendet wird. So kommen keine veralteten Nachrichten an, wenn die Pflanze zwischenzeitlich gegossen wurde.

**Pumpen-Sicherheit**: Automatisches Gießen nutzt `wait_for_trigger` für die Pumpen-Laufzeit statt `delay:`, damit bei `mode: restart` immer ein `turn_off` direkt nach dem Wait ausgeführt wird — egal ob Timer oder Trigger den Wait beendet hat. (`delay:` ließ die Pumpe bei gleichzeitigem Trocken-Trigger offen laufen, behoben in v2.4.)

**Benachrichtigungs-Deduplizierung**: Jede Automationsinstanz bekommt eine eindeutige `notification_id` aus `sensor | slugify`. Diese wird als `tag:` in Mobile-Push und als ID bei `persistent_notification` verwendet — wiederholte Trigger aktualisieren so die bestehende Benachrichtigung, statt eine neue zu erzeugen.

**Cooldown-Guard**: `cooldown_ok` ist eine Template-Variable, berechnet aus `this.attributes.last_triggered`. Sie wird pro Trigger ausgewertet, nicht als Trigger-Bedingung selbst.

## Bekannte Fallstricke (aus dem Changelog)

Diese Fehler sind in der Vergangenheit aufgetreten — nicht wiederholen:

| Problem | Ursache | Lösung |
|---|---|---|
| Pumpe läuft immer volle Zeit | `delay:` statt `wait_for_trigger` | `wait_for_trigger` mit sofortigem `turn_off` danach |
| Alexa sprach nie | `person_in_zone` fehlte im `variables:`-Block | Immer alle verwendeten Variablen in `variables:` deklarieren |
| Mobile-Benachrichtigung kam nie an | `tag:` auf falscher Einrückungsebene in `data:` | `tag:` gehört in `data.data:`, nicht `data:` |
| Iterationsschleife über Buchstaben | `alexa_targets=false` wurde direkt iteriert | Immer `alexa_targets_safe` verwenden statt rohem `alexa_targets` |
| Laufzeitfehler bei optionalen Feldern | `default: false` bei Entity-Feldern | `default: null` für optionale Entity/Device-Felder |
| `states(false)` Fehler | `is_paused` rief `states()` auf wenn kein Pause-Entity gesetzt | Null-Guard vor `states()`-Aufruf |

## Validierung & HA-Workflow

**YAML-Syntax lokal prüfen:**
```bash
yamllint Bodenfeuchte.yaml
```

**Jinja2-Templates in HA testen:**  
Entwicklerwerkzeuge → Template (linke Seitenleiste) → Template einfügen und mit realen Entity-IDs auswerten.

**Blueprint neu importieren nach Änderungen:**  
Einstellungen → Automationen → Blueprints → Blueprint-Karte → drei Punkte → „Blueprint neu laden". Bestehende Automationen, die diesen Blueprint verwenden, übernehmen die Änderungen beim nächsten Trigger automatisch.

**Automation manuell auslösen:**  
In der Automation-Detailansicht oben rechts auf „Ausführen" klicken — simuliert einen Trigger ohne auf den Sensor warten zu müssen.

## Versionierung

Version steht in `blueprint.name` und in `blueprint.description` (der Description enthält den vollständigen Changelog in `<details>`-Blöcken). Die Datei heißt immer `Bodenfeuchte.yaml` — kein Versionssuffix im Dateinamen, damit die `source_url` stabil bleibt.

---

# Alexa Timer → AWTRIX 3 (`alexa_timer_awtrix.yaml`)

Zeigt den nächsten Alexa-Timer (aus [Alexa Media Player](https://github.com/alandtse/alexa_media_player), HACS) als Live-Countdown auf einem [AWTRIX 3 / awtrix-light](https://blueforcer.github.io/awtrix3/)-Display (Ulanzi). Bei Ablauf erscheint ein Alarmtext (Standard `!!!LOS!!!`). Ersetzt einen früheren Aufbau aus 2 Template-Sensoren + 4 Automationen durch **eine** Automation, **ohne** Helfer/Template-Sensor.

## Datenfluss & Anzeige

- **Quelle:** der STATE von `sensor.<gerät>_next_timer`. Ausgabe per **MQTT** an AWTRIX.
- **Anzeige-Modus** (Input `display_mode`):
  - `app` → Custom App, Topic `<prefix>/custom/<app_name>`. Löschen = leerer Payload an dasselbe Topic.
  - `pinned` → gehaltene Notification (`hold:true`, `stack:false`), Topic `<prefix>/notify`. Löschen = leerer Payload an `<prefix>/notify/dismiss`.
- **Farben:** `color_rgb`-Selector → `[r,g,b]`-Array; AWTRIX akzeptiert das Array direkt (`{{ farbe | to_json }}`).
- **Alarm-Effekt:** `select`-Dropdown mit den gültigen awtrix-light-Effektnamen + „Kein Effekt" (`none`).

## ⚠️ Alexa-Media-Player-Sensor: die zentralen Eigenheiten

Das Timer-Datenmodell von AMP ist unzuverlässig — **darauf gründet das gesamte Design**:

- Bei **laufendem** Timer ist der **STATE** ein zukünftiger ISO-Zeitstempel (= echte Zielzeit). **Das ist die einzige verlässliche Quelle.**
- Bei **keinem** Timer / nach Dismiss liefert der STATE Müll ≈ jetzt oder `unavailable`.
- Beim **Klingeln** „klebt" der STATE auf ≈ jetzt und ändert sich erst, wenn der Timer auf Alexa beendet/quittiert wird.
- Das Attribut **`active`** ist oft `None`/leer — **nicht** verwenden. Abgelaufene Timer bleiben als „ON"-Leichen mit vergangener `triggerTime` hängen.

## Architektur-Muster

**Kein Sekunden-Trigger.** Trigger sind nur `state` (Quell-Sensor) + `homeassistant: start`. Der Sekundentakt entsteht in einer `repeat: while`-Schleife, die **nur während eines aktiven Timers** läuft → im Leerlauf null Last. (Ein früherer `time_pattern: /1` feuerte 86 400×/Tag und belastete das System.)

**`mode: single`** (nicht `restart`!): Beim Klingeln ändert der Sensor seinen State; mit `restart` würde das die laufende Countdown-Schleife abbrechen. `single` lässt die Schleife sauber bis zur Zielzeit durchlaufen.

**Zielzeit einfrieren (`locked_target`).** Top-Level-`variables:` mit `now()`/State werden im Schleifen-Kontext faktisch live neu ausgewertet → die Schleife „klebte" bei `00:00`. Lösung: die Zielzeit am Schleifenanfang per **`- variables:`-Action-Schritt** als Zahl einfrieren und die Schleife dagegen zählen:
```yaml
- variables:
    locked_target: "{{ target_epoch | float(0) }}"   # einmalig, statisch
- repeat:
    while:
      - "{{ locked_target - now().timestamp() > 0 }}"
```

**Alarm rein zeitbasiert.** Der Alarm wird **ausschließlich** ausgelöst, wenn die Schleife die eingefrorene Zielzeit erreicht (innere `choose:` nach der Schleife). Er wird **nie** aus einem Sensor-Statuswechsel abgeleitet → kein Fehlalarm beim Stellen/Abbrechen. Endet die Schleife vorzeitig (Sensor wird ungültig = Abbruch), wird stattdessen nur die Anzeige gelöscht.

**UTC-Epoch-Vergleiche.** Zielzeit und „jetzt" werden über `.timestamp()` (UTC-Epoch) verglichen — zeitzonensicher, kein naiv/aware-Mischvergleich.

## Bekannte Fallstricke (aus der Entwicklung)

| Problem | Ursache | Lösung |
|---|---|---|
| Sofort `!!!LOS!!!` beim Stellen | Ablauf wurde aus dem Sensor-Statuswechsel abgeleitet; Müll/„ON"-Leichen ≈ jetzt galten als „abgelaufen" | Alarm NUR zeitbasiert beim Schleifenende; Sensorwechsel nie als Ablauf werten |
| Countdown „klebt" bei `00:00`, LOS erst beim Beenden | gemerkte Zielzeit wurde im Schleifen-Kontext live neu berechnet | Zielzeit per `- variables:`-Schritt als Zahl einfrieren (`locked_target`) |
| Hohe Dauerlast im Leerlauf | `time_pattern: /1`-Trigger (86 400 Läufe/Tag) | Sekundentakt in `repeat: while` nur während aktivem Timer; kein Sekunden-Trigger |
| Schleife bricht beim Klingeln ab | `mode: restart` + Sensor-Statuswechsel beim Klingeln | `mode: single` |
| `active`-Attribut leer/None | AMP füllt `active` unzuverlässig | STATE als Quelle nutzen, keine Attribute |
| Blueprint lädt nicht: „expected str … Got None" | `select`-Optionen mischten Strings und `{label,value}` | alle Optionen einheitlich als `{label,value}` |
| Effekt wirkte nicht wie beschrieben | freies Textfeld + fest verdrahtete `effectSettings`/`palette` | `select`-Dropdown gültiger Effektnamen; nur `"effect"` senden |
| Zeitzonen-Verschiebung im Countdown | `as_datetime`-Vergleich, `timestamp_custom` | in UTC-Epoch (`.timestamp()`) rechnen |

**Pinned-Modus-Vorbehalt:** Beim regulären Ablauf ersetzt der LOS-Alarm den gehaltenen Countdown automatisch (`stack:false`). Meldet Alexa den Timer danach als entfernt, kann der `dismiss`-Pfad den LOS im Pinned-Modus verkürzen. Im `app`-Modus bleibt der LOS (eigene Notification) erhalten, nur die Countdown-App wird entfernt.

## Diagnose

Bei Timer-Problemen zuerst den Quell-Sensor **bei laufendem Timer** prüfen (Entwicklerwerkzeuge → Vorlage):
```jinja2
{% set src = 'sensor.alexa_wohnzimmer_next_timer' %}
STATE:          {{ states(src) }}
Sek. bis STATE: {{ (as_datetime(states(src)) - now()).total_seconds() if as_datetime(states(src)) else 'n/a' }}
```
`Sek. bis STATE` muss bei laufendem Timer **positiv** sein (= echte Restzeit). Ist sie 0/negativ, liefert AMP Müll → das ist der Knackpunkt.

Bei „LOS kommt nicht/zu spät": Trace des langen Countdown-Laufs ansehen — die `while`-Auswertungen zeigen, ob `locked_target` korrekt eingefroren ist und gegen `now()` zählt.
