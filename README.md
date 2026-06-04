# Home Assistant Blueprints

Eine Sammlung von [Home Assistant](https://www.home-assistant.io/) Blueprints
rund um **AWTRIX 3** (Ulanzi-Uhr) und Pflanzenpflege. Jede Blueprint lässt sich
mit einem Klick über den **„In Home Assistant öffnen"**-Button importieren oder
manuell per URL.

## Inhaltsverzeichnis

- [Alexa Timer → AWTRIX 3](#alexa-timer--awtrix-3)
- [AWTRIX Wetter-Overlay (OpenWeatherMap)](#awtrix-wetter-overlay-openweathermap)
- [Bodenfeuchte Alarm System](#bodenfeuchte-alarm-system)
- [Installation & Updates](#installation--updates)

| Blueprint | Kurzbeschreibung |
|---|---|
| [🔔 Alexa Timer → AWTRIX 3](#alexa-timer--awtrix-3) | Alexa-Timer als Live-Countdown auf dem AWTRIX-Display + Alarm bei Ablauf |
| [🌦️ AWTRIX Wetter-Overlay](#awtrix-wetter-overlay-openweathermap) | Setzt passend zum Wetter ein AWTRIX-Overlay (Regen, Schnee, Gewitter …) |
| [🌱 Bodenfeuchte Alarm System](#bodenfeuchte-alarm-system) | Trocken-Alarm, Auto-Gießen und Multi-Channel-Benachrichtigung für Pflanzen |

---

## Alexa Timer → AWTRIX 3

**Datei:** `alexa_timer_awtrix.yaml`

Zeigt den nächsten Alexa-Timer als Live-Countdown auf einem AWTRIX-3-Display
(Ulanzi / awtrix-light) an und löst beim Ablauf einen Alarm aus (Standard
`!!!LOS!!!`). Ersetzt den klassischen Aufbau aus mehreren Template-Sensoren +
Automationen durch **eine einzige Automation** – ohne separaten Sensor, ohne
Helfer.

**Funktionen**

- Live-Countdown – läuft in einer Schleife **nur während eines aktiven Timers**,
  kein Sekunden-Trigger → **im Leerlauf null Systemlast**
- Anzeige-Modus wählbar: **Im App-Wechsel** (Custom App) oder **Nur Timer
  anpinnen** (gehaltene Notification)
- Farben per Farbrad, Alarm-Effekt als Dropdown, optionales Icon
- Blinken & Alarmdauer einstellbar
- **Fehlalarm-sicher:** Alarm nur beim Erreichen der Zielzeit – nie beim Stellen
  oder Abbrechen; Abbruch entfernt die Anzeige automatisch

**Voraussetzungen:** MQTT + AWTRIX 3, [Alexa Media Player](https://github.com/alandtse/alexa_media_player) (HACS)

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Falexa_timer_awtrix.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/alexa_timer_awtrix.yaml
```

---

## AWTRIX Wetter-Overlay (OpenWeatherMap)

**Datei:** `overlay_weatherV1.yaml`

Setzt automatisch ein passendes **Overlay** auf der AWTRIX-Anzeige, sobald sich
der OpenWeatherMap-Wettercode ändert – z. B. Regentropfen bei Regen oder
Schneeflocken bei Schnee. Das Overlay wird per MQTT in die AWTRIX-Settings
geschrieben (`OVERLAY`-Key).

**Funktionen**

- Mappt OpenWeatherMap-Wettercodes auf AWTRIX-Overlays:
  `thunder`, `drizzle`, `rain`, `storm`, `snow`, `frost`, `clear`
- Starkregen (502–504, 522), Tornado/Böen (781, 771) → `storm`
- Nebel/Dunst/Sand (7xx) → `frost`; klarer/bewölkter Himmel (800–804) → `clear`

**Inputs**

- **Wettercode-Sensor** – z. B. `sensor.openweathermap_weather_code`
- **MQTT-Topic** – Settings-Topic der AWTRIX (Standard `awtrix/settings`)

**Voraussetzungen:** MQTT + AWTRIX 3, [OpenWeatherMap](https://www.home-assistant.io/integrations/openweathermap/)-Integration mit Wettercode-Sensor

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Foverlay_weatherV1.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/overlay_weatherV1.yaml
```

---

## Bodenfeuchte Alarm System

**Datei:** `Bodenfeuchte.yaml`

Überwacht einen Bodenfeuchte-Sensor und benachrichtigt zuverlässig, sobald eine
Pflanze Wasser braucht – und wieder, wenn sie versorgt ist. Optional gießt sie
automatisch. Alles über Einstellungen konfigurierbar, kein Code nötig.

**Funktionen**

- **Trocken-Alarm** mit Grenzwert, Hysterese, Verzögerung und Cooldown-Schutz
- **OK-Rückmeldung** („Alles gut"), wenn die Feuchte wieder stabil ist
- **Benachrichtigungskanäle** einzeln schaltbar: 📱 Mobile App (mehrere Geräte),
  📋 anhaltende HA-Benachrichtigung, 🔊 Alexa-Durchsage (mit Lautstärkesteuerung)
- **Anwesenheitserkennung** – Alexa nur, wenn jemand zuhause ist
- **Nachtmodus** pro Kanal (auch über Mitternacht) und **Urlaubs-/Pause-Modus**
- **Automatisches Gießen** – Pumpe/Ventil mit Laufzeit & Sicherheitsabschaltung,
  optional zeitplanbasiertes präventives Gießen

**Voraussetzungen:** Bodenfeuchte-Sensor (%); optional Pumpe/Ventil (`switch.*`),
Mobile-App-Benachrichtigung, Alexa

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2FBodenfeuchte.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/Bodenfeuchte.yaml
```

Eigenständiges Repo: [x1marc/Bodenfeuchte-Alarm-System](https://github.com/x1marc/Bodenfeuchte-Alarm-System)

---

## Installation & Updates

**Import:** Auf den **„In Home Assistant öffnen"**-Button der jeweiligen
Blueprint klicken – HA öffnet den Import-Dialog mit vorausgefüllter URL.
Alternativ in HA unter **Einstellungen → Automationen & Szenen → Blueprints →
Blueprint importieren** die jeweilige URL einfügen.

**Update:** Nach Änderungen an einer Blueprint in der Blueprint-Übersicht auf die
drei Punkte → **„Blueprint neu laden"**. Bestehende Automationen übernehmen die
Änderungen beim nächsten Auslösen automatisch.

**Voraussetzung für die AWTRIX-Blueprints:** aktive MQTT-Integration und eine per
MQTT verbundene AWTRIX-3-Uhr ([awtrix-light](https://blueforcer.github.io/awtrix3/)).
