# Home Assistant Blueprints

Sammlung von [Home Assistant](https://www.home-assistant.io/) Blueprints.

| Blueprint | Zweck |
|---|---|
| [Alexa Timer → AWTRIX 3](#alexa-timer--awtrix-3) | Alexa-Timer als Live-Countdown auf einem AWTRIX-3-Display + Alarm bei Ablauf |
| `Bodenfeuchte.yaml` | Bodenfeuchte-Alarm mit Zeitplan-Gießen, Pumpensteuerung und Multi-Channel-Benachrichtigung |

---

## Alexa Timer → AWTRIX 3

Zeigt den nächsten Alexa-Timer als Live-Countdown auf einem AWTRIX-3-Display
(Ulanzi / awtrix-light) an und löst beim Ablauf einen Alarm aus (Standard
`!!!LOS!!!`).

Ersetzt den klassischen Aufbau aus mehreren Template-Sensoren + Automationen
durch **eine einzige Automation** – kein separater Sensor, kein Helfer, keine
`homeassistant.update_entity`-Schleife.

**Funktionen**

- Live-Countdown im Sekundentakt – läuft in einer Schleife **nur während eines
  aktiven Timers**. Kein Sekunden-Trigger → **im Leerlauf null Systemlast**.
- Anzeige-Modus wählbar:
  - **Im App-Wechsel** – als Custom App, rotiert mit den übrigen AWTRIX-Apps
  - **Nur Timer anpinnen** – gehaltene Notification, überdeckt alle anderen Apps
- Farben per Farbrad (`color_rgb`)
- Alarm-Effekt als Dropdown (gültige AWTRIX-Effektnamen) + optionales Icon
- Blinken des Alarm-Textes optional, Alarmdauer einstellbar
- **Fehlalarm-sicher:** Der Alarm wird ausschließlich beim Erreichen der
  Zielzeit ausgelöst – nie beim Stellen, Abbrechen oder bei Sensor-Macken.
- **Abbruch** auf Alexa entfernt die Anzeige automatisch.

**Voraussetzungen**

- MQTT-Integration aktiv, AWTRIX 3 (awtrix-light) per MQTT verbunden
- [Alexa Media Player](https://github.com/alandtse/alexa_media_player) (HACS)
  für den `*_next_timer`-Sensor

**Import**

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Falexa_timer_awtrix.yaml)

Oder manuell: **Einstellungen → Automationen & Szenen → Blueprints → Blueprint
importieren** und folgende URL einfügen:

```
https://github.com/x1marc/ha-blueprints/blob/main/alexa_timer_awtrix.yaml
```

> **Hinweis zur Quelle:** Alexa Media Player liefert Timer-Daten unzuverlässig
> (das `active`-Attribut ist oft leer, abgelaufene Timer „kleben"). Diese
> Blueprint nutzt deshalb ausschließlich den STATE des Sensors und merkt sich
> die echte Zielzeit beim Start der Countdown-Schleife. Details siehe
> [`CLAUDE.md`](CLAUDE.md).
