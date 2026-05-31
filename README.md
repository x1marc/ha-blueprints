# Home Assistant Blueprints

Sammlung von [Home Assistant](https://www.home-assistant.io/) Blueprints.

## Alexa Timer → AWTRIX 3

Zeigt den nächsten Alexa-Timer als Live-Countdown auf einem AWTRIX-3-Display
(Ulanzi / awtrix-light) an und löst beim Ablauf einen Alarm aus.

Ersetzt den klassischen Aufbau aus mehreren Template-Sensoren + Automationen
durch **eine einzige Automation** (der Countdown wird inline berechnet, kein
separater Sensor, keine `homeassistant.update_entity`-Schleife).

**Funktionen**

- Live-Countdown im Sekundentakt (`time_pattern`, `mode: restart`)
- Anzeige-Modus wählbar:
  - **Im App-Wechsel** – als Custom App, rotiert mit den übrigen AWTRIX-Apps
  - **Nur Timer anpinnen** – gehaltene Notification, überdeckt alle anderen Apps
- Farben per Farbrad (`color_rgb`)
- Alarm-Effekt als Dropdown (gültige AWTRIX-Effektnamen)
- Blinken des Alarm-Textes optional
- Fehlalarm-sicher: Alarm feuert nur bei **echtem** Ablauf (Prüfung des
  vorherigen Zeitstempels), nicht bei Sensor-Flackern oder beim Timer-Stellen

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
