# Home Assistant Blueprints

<p align="center">
  <b>☕ Gefällt dir dieses Projekt? Dann spendier mir gern einen Kaffee!</b><br><br>
  <a href="https://buymeacoffee.com/x1marc"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" height="60"></a>
</p>

Eine Sammlung von [Home Assistant](https://www.home-assistant.io/) Blueprints
rund um **AWTRIX 3 / AWTRIX NG** (Ulanzi-Uhr) und Pflanzenpflege. Jede Blueprint lässt sich
mit einem Klick über den **„In Home Assistant öffnen"**-Button importieren oder
manuell per URL.

## Inhaltsverzeichnis

- [Alexa Timer → AWTRIX 3 & NG](#alexa-timer--awtrix-3--ng)
- [AWTRIX Wetter-Overlay (OpenWeatherMap)](#awtrix-wetter-overlay-openweathermap)
- [Sensor → AWTRIX NG App](#sensor--awtrix-ng-app)
- [Bodenfeuchte Alarm System](#bodenfeuchte-alarm-system)
- [Installation & Updates](#installation--updates)

| Blueprint | Kurzbeschreibung |
|---|---|
| [🔔 Alexa Timer → AWTRIX 3 & NG](#alexa-timer--awtrix-3--ng) | Alexa-Timer als Live-Countdown auf mehreren AWTRIX-Uhren (Firmware 3 **und** NG) + Alarm bei Ablauf |
| [🌦️ AWTRIX Wetter-Overlay](#awtrix-wetter-overlay-openweathermap) | Setzt passend zum Wetter ein AWTRIX-Overlay (Regen, Schnee, Gewitter …) |
| [📈 Sensor → AWTRIX NG App](#sensor--awtrix-ng-app) | Zeigt jeden Sensorwert als eigene App auf AWTRIX NG – **ohne eigene Automation** |
| [🌱 Bodenfeuchte Alarm System](#bodenfeuchte-alarm-system) | Trocken-Alarm, Auto-Gießen und Multi-Channel-Benachrichtigung für Pflanzen |

---

## Alexa Timer → AWTRIX 3 & NG

**Datei:** `alexa_timer_awtrix.yaml`

Zeigt den nächsten Alexa-Timer als Live-Countdown auf **einer oder mehreren**
AWTRIX-Uhren an und löst beim Ablauf einen Alarm aus (Standard `!!!LOS!!!`).
Unterstützt **beide Firmwares gleichzeitig** – [AWTRIX 3 / awtrix-light](https://blueforcer.github.io/awtrix3/)
und [AWTRIX NG](https://blueforcer.github.io/awtrix-ng/). Ersetzt den klassischen
Aufbau aus mehreren Template-Sensoren + Automationen durch **eine einzige
Automation** – ohne separaten Sensor, ohne Helfer.

**Funktionen**

- **Mehrere Geräte & beide Firmwares:** je eine Liste für „AWTRIX 3 Geräte" und
  „AWTRIX NG Geräte" – beliebig viele, beliebig gemischt. Die Firmware wählst du
  durch die Liste, in die du das Gerät einträgst (unterschiedliche MQTT-Topics
  & Payloads werden automatisch verwendet).
- Live-Countdown – läuft in einer Schleife **nur während eines aktiven Timers**,
  kein Sekunden-Trigger → **im Leerlauf null Systemlast**
- Anzeige-Modus wählbar: **Im App-Wechsel** (Custom/Pushed App) oder **Nur Timer
  anpinnen** (gehaltene Notification)
- Farben per Farbrad, Alarm-Effekt als Dropdown, optionales Icon
- Blinken & Alarmdauer einstellbar
- **Fehlalarm-sicher:** Alarm nur beim Erreichen der Zielzeit – nie beim Stellen
  oder Abbrechen; Abbruch entfernt die Anzeige auf allen Geräten

**Voraussetzungen:** MQTT + mind. eine AWTRIX-Uhr (Firmware 3 oder NG),
[Alexa Media Player](https://github.com/alandtse/alexa_media_player) (HACS)

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Falexa_timer_awtrix.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/alexa_timer_awtrix.yaml
```

---

## AWTRIX Wetter-Overlay (OpenWeatherMap)

**Datei:** `overlay_weatherV1.yaml`

Setzt automatisch ein passendes **Overlay** auf **einer oder mehreren**
AWTRIX-Uhren, sobald sich der OpenWeatherMap-Wettercode ändert (und beim
HA-Start) – z. B. Regentropfen bei Regen oder Schneeflocken bei Schnee.
Unterstützt **beide Firmwares gleichzeitig**:
[AWTRIX 3](https://blueforcer.github.io/awtrix3/) (`<prefix>/settings`, `OVERLAY`)
und [AWTRIX NG](https://blueforcer.github.io/awtrix-ng/) (`<prefix>/cmd/display`, `overlay`).

**Funktionen**

- Mappt OpenWeatherMap-Wettercodes auf AWTRIX-Overlays:
  `thunder`, `drizzle`, `rain`, `storm`, `snow`, `frost`, `clear`
- Starkregen (502–504, 522), Tornado/Böen (781, 771) → `storm`
- Nebel/Dunst/Sand (7xx) → `frost`; klarer/bewölkter Himmel (800–804) → `clear`
- **Mehrere Geräte & beide Firmwares** über zwei Listen; bei NG wird `clear`
  automatisch als `overlay: null` gesendet (Overlay aus)
- Setzt das Overlay auch **nach HA-Neustart** einmalig

**Inputs**

- **Wettercode-Sensor** – z. B. `sensor.openweathermap_weather_code`
- **AWTRIX 3 Geräte** / **AWTRIX NG Geräte** – je eine Liste von Topic-Prefixen

**Voraussetzungen:** MQTT + mind. eine AWTRIX-Uhr (Firmware 3 oder NG),
[OpenWeatherMap](https://www.home-assistant.io/integrations/openweathermap/)-Integration mit Wettercode-Sensor

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Foverlay_weatherV1.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/overlay_weatherV1.yaml
```

---

## Sensor → AWTRIX NG App

**Datei:** `sensor_to_awtrix_ng_app.yaml`

Zeigt den Wert eines **beliebigen Sensors** als eigene App auf einer
**AWTRIX-NG**-Uhr ([blueforcer/awtrix-ng](https://blueforcer.github.io/awtrix-ng/))
und hält ihn automatisch aktuell – **ohne dass du eine Automation schreiben
musst**. Importieren, Sensor wählen, App benennen, fertig. (Nur AWTRIX NG, nicht
AWTRIX 3.)

**Funktionen**

- Anzeige-Text wird komfortabel gebaut: **[Text davor] [Wert gerundet] [Einheit]**
  → z. B. `PV 12.3 kWh`. Nicht-numerische Werte (z. B. `Zuhause`) werden
  unverändert angezeigt.
- **Aktualisierung** bei jeder Sensoränderung, in einem wählbaren Intervall
  (5–30 Min) und nach HA-Neustart. Gleicher App-Name → App wird überschrieben.
- **Icon** (Nummer) + Icon-Modus, **Farbe** per Farbrad, **Anzeigedauer** und
  optionales **Selbst-Löschen** (Lifetime)
- Sendet **direkt per MQTT** – die
  [HA-AWTRIX-NG-Skripte](https://github.com/x1marc/ha-awtrix-ng-scripts) werden
  dafür **nicht** benötigt

**Inputs (Auszug):** Sensor · App-Name · Text davor / Einheit / Nachkommastellen ·
Icon + Modus · Textfarbe · Intervall · Dauer · Lifetime · Prefix (Standard `awtrixng`)

**Voraussetzungen:** MQTT + eine per MQTT verbundene **AWTRIX-NG**-Uhr

[![In Home Assistant öffnen](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fx1marc%2Fha-blueprints%2Fblob%2Fmain%2Fsensor_to_awtrix_ng_app.yaml)

```
https://github.com/x1marc/ha-blueprints/blob/main/sensor_to_awtrix_ng_app.yaml
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
