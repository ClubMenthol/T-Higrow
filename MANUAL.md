# T-Higrow Cannabis Monitor
Vollständige Dokumentation & Kalibrierungsanleitung

**Version 4.9.6**  
*ESPHome Projekt für LILYGO T-Higrow ESP32 Board*  
Ein intelligenter Multisensor für präzise Pflanzenbau-Überwachung  
**Letztes Update:** August 2026

---

## Inhaltsverzeichnis
1. [Projektübersicht](#1-projektübersicht)
2. [Hardware-Setup & Installation](#2-hardware-setup--installation)
3. [Erste Konfiguration (secrets.yaml)](#3-erste-konfiguration-secretsyaml)
4. [Modul-Referenz](#4-modul-referenz)
5. [Sensoren verstehen](#5-sensoren-verstehen)
6. [Kalibrierung Schritt-für-Schritt](#6-kalibrierung-schritt-für-schritt)
7. [VPD & Klima-Konzepte](#7-vpd--klima-konzepte)
8. [Fehlerbehandlung & Troubleshooting](#8-fehlerbehandlung--troubleshooting)
9. [Häufig gestellte Fragen (FAQ)](#9-häufig-gestellte-fragen-faq)

---

## 1. Projektübersicht

### 1.1 Was ist der T-Higrow Cannabis Monitor?
Der T-Higrow Cannabis Monitor ist ein umfassendes ESPHome-Projekt für das LILYGO T-Higrow ESP32-Board. Es misst und überwacht alle kritischen Umweltparameter für den optimalen Pflanzenbau:
- ✓ Bodenfeuchte (kapazitiv, 0–100%)
- ✓ Elektrische Leitfähigkeit / Düngergehalt (0–5 mS/cm)
- ✓ Temperatur & Luftfeuchtigkeit (DHT11)
- ✓ Lichtintensität (BH1750 Sensor, Lux)
- ✓ Akkuspannung & Ladestatus (18650 Lithium-Zelle)
- ✓ VPD (Vapor Pressure Deficit) & Schimmel-Risiko
- ✓ DLI-Akkumulation (Daily Light Integral)
- ✓ Taupunkt & Hitzestress-Tracking

### 1.2 Kernfeatures
- 🔹 **Modulare Architektur:** Einzelne Funktionen können ein-/ausgeschaltet werden
- 🔹 **5-Punkt-Kalibrierung:** Hochpräzise EC-Messung mit stückweise linearer Interpolation
- 🔹 **Mehrschicht-Filterung:** Median- + Moving-Average-Filter für stabile Werte
- 🔹 **Deep Sleep:** Automatischer Standby nach 15 Min. (anpassbar)
- 🔹 **MQTT & Home Assistant Integration**
- 🔹 **Wlan-Fallback & Notfall-Timeout** (bei MQTT-Ausfall)
- 🔹 **MAC-basiertes Pflanzenmapping:** Mehrere Boards automatisch erkannt

### 1.3 Technische Spezifikationen
- **Hardware:** LILYGO T-Higrow ESP32 Board (V1.1 mit DHT11)
- **Stromversorgung:** 18650 Lithium-Zelle oder USB-C (Micro)
- **Sensoren:** ADC (Feuchte, EC, Akku), I2C (BH1750, DHT11)
- **Betriebsmodus:** Deep Sleep (15 Min. Standard) oder Dauerbetrieb
- **Kommunikation:** WLAN (2.4 GHz), MQTT, Home Assistant API

### 1.4 Versionshistorie
- **4.9.6:** Fallback-Timeout für MQTT-Ausfälle, globale DLI-Variable
- **4.9.5:** BSSID-Mapping in `secrets.yaml` ausgelagert (Datenschutz)
- **4.9.2–4.9.3:** Filter-Fixes, EC-Kalibrierung optimiert

---

## 2. Hardware-Setup & Installation

### 2.1 Benötigte Komponenten
- 1x LILYGO T-Higrow ESP32 Board
- 1x 18650 Lithium-Ionen Zelle (optional für kabellos)
- 1x Bodenfeuchte-Sensor (kapazitiv, on-board)
- 1x EC/Leitfähigkeits-Sensor (on-board)
- 1x DHT11 Temperatur/Feuchte-Sensor (on-board)
- 1x BH1750 Lichtsensor (I2C, on-board)
- 1x USB-C Kabel (für Programmierung & optional Stromversorgung)

> 💡 **Hinweis:** Das Board ist vollständig mit allen Sensoren bestückt. Keine zusätzliche Hardware nötig!

### 2.2 Montage & Platzierung
**Platzierung des Boards:**
- Bodenfeuchte-Sensor direkt im Topf / Substrat (oben einführen)
- EC-Sensor ebenfalls im Topf (gleiche Position wie Feuchte-Sensor)
- DHT11 & BH1750 sollten im Blattbereich angebracht sein (nicht im Schatten)
- Board selbst möglichst zentral, trocken und vor direktem Licht geschützt

**Stromversorgung:**
- **Akkumodus:** 18650 Zelle in das Fach einlegen → Automatischer Deep Sleep nach 15 Min.
- **USB-Modus:** USB-C Kabel anstecken → Dauerbetrieb

### 2.3 ESPHome Flashen
**Schritt 1: ESPHome installieren**
https://esphome.io/guides/installing_esphome.html
*(Docker-Version empfohlen für cross-platform)*

**Schritt 2: Projekt-Dateien herunterladen**
GitHub: https://github.com/ClubMenthol/T-Higrow
→ Gesamtes Repository klonen oder als ZIP herunterladen

**Schritt 3: Ordner-Struktur**
```text
esphome/
├── t-higrow-main.yaml
├── secrets.yaml (← ERSTELLEN!)
└── modules/
    ├── system.yaml
    ├── klima.yaml
    ├── boden.yaml
    ├── energie.yaml
    ├── erweitert.yaml
    └── pflanzen_mapping.yaml
```

**Schritt 4: Secrets-Datei erstellen**
→ Siehe Abschnitt 3 (Konfiguration)

**Schritt 5: Flashen**
```bash
esphome run t-higrow-main.yaml
```
→ Board auswählen → USB-Verbindung bestätigen → Warten (~2–5 Min.)

**Schritt 6: Verbindung testen**
- Home Assistant → Integrations → ESPHome → Gerät sollte auftauchen
- Oder: Webinterface: `http://<IP-Adresse>:80`

---

## 3. Erste Konfiguration (secrets.yaml)

### 3.1 secrets.yaml Vorlage
Erstelle im ESPHome-Ordner eine neue Datei namens `secrets.yaml` und füge folgende Inhalte ein. Ersetze die Werte durch deine eigenen:

```yaml
# Geräte-Namen
devicename: "t-higrow-001"
friendly_name: "Pflanze 1"

# WLAN-Konfiguration
wifi_ssid: "MeinWLAN"
wifi_password: "geheimespasswort123"
wifi_bssid: "AA:BB:CC:DD:EE:FF"  # Optional für schnellere Verbindung

# Statische IP (optional aber empfohlen)
manual_static_ip: "192.168.178.100"
manual_gateway: "192.168.178.1"
manual_subnet: "255.255.255.0"
manual_dns1: "8.8.8.8"
manual_dns2: "8.8.4.4"

# ESPHome API-Verschlüsselung
api_encryption_key: "GENERIERE_MIT: esphome generate-key"
# Command:
# esphome generate-key

# MQTT-Broker (optional)
mqtt_broker: "192.168.178.50"

# Bodenfeuchte-Kalibrierung (siehe Abschnitt 6)
soil_moist_raw_dry: "2400"
soil_moist_raw_wet: "1200"

# EC-Kalibrierung (5-Punkt, siehe Abschnitt 6)
soil_ec_raw_p0: "0"
soil_ec_target_p0: "0.0"
soil_ec_raw_p1: "200"
soil_ec_target_p1: "0.3"
soil_ec_raw_p2: "500"
soil_ec_target_p2: "0.8"
soil_ec_raw_p3: "900"
soil_ec_target_p3: "1.5"
soil_ec_raw_p4: "3000"
soil_ec_target_p4: "5.0"

# MAC-Adresse & Pflanzenmapping
mac_p1: "a1b2c3d4e5f6"
name_p1: "Cannabis - Fensterbank"
mac_p2: "f6e5d4c3b2a1"
name_p2: "Cannabis - Growbox"

# WLAN Access Point Mapping (BSSID → Name)
bssid_mapping_logic: |-
  if (id(wifi_bssid_sensor).state == "AA:BB:CC:DD:EE:FF") return {"Wohnzimmer"};
  if (id(wifi_bssid_sensor).state == "FF:EE:DD:CC:BB:AA") return {"Keller"};
  return {"Unbekanntes Netzwerk"};
```

### 3.2 Wichtige Hinweise
🔒 **Sicherheit:**
- Gib die `secrets.yaml` NIEMALS an öffentliche Repositories oder GitHub!
- `.gitignore` sollte `secrets.yaml` enthalten

📍 **MAC-Adresse herausfinden:**
```bash
esphome logs t-higrow-main.yaml
```
→ Nach "MAC Address" suchen

🔑 **API-Schlüssel generieren:**
```bash
esphome generate-key
```

📡 **WLAN-BSSID auslesen:**
Auf Windows/Linux: `nmcli device wifi`
Format: `AA:BB:CC:DD:EE:FF` (ohne Leerzeichen)

---

## 4. Modul-Referenz

### 4.1 Modul: BODEN (`boden.yaml`)
**Messungen:**
- Boden: 1 Feuchte (0–100%)
- Boden: 2 Status (Zu trocken / Optimal / Sehr nass)
- Boden: 3 Leitfähigkeit/EC (0–5 mS/cm)
- Boden: 4 Dünger-Index (Zu wenig / Optimal / Überdüngt)

**Wie es funktioniert:**
1. Der kapazitive Feuchte-Sensor auf Pin 32 misst die Bodenfeuchte als Rohwert (0–4095)
2. Dieser wird gegen die Kalibrierungswerte (trocken/nass) in % umgerechnet
3. Ein Median-Filter (5 Werte) entfernt Ausreißer
4. Ein Moving-Average-Filter (30 Werte) glättet die Kurve alle 60 Sekunden
5. Ein Text-Sensor vergleicht den Wert mit Schwellenwerten

**EC-Sensor (Dünger):**
- 5-Punkt-Kalibrierung (linear interpoliert)
- P0: 0,0 mS/cm (Regenwasser)
- P1: 0,3 mS/cm (nährstoffarm)
- P2: 0,8 mS/cm (Anzuchterde)
- P3: 1,5 mS/cm (optimal)
- P4: 5,0 mS/cm (Salzlösung maximal)

**Schwellenwerte (in `t-higrow-main.yaml` anpassen):**
```yaml
soil_moisture_dry: "30.0"   # Alarm unter 30%
soil_moisture_wet: "80.0"   # Warnung über 80%
soil_ec_low: "0.6"          # Zu wenig Nährstoffe
soil_ec_high: "2.2"         # Überdüngung
```

### 4.2 Modul: KLIMA (`klima.yaml`)
**Messungen:**
- Klima: 1 Temperatur (°C)
- Klima: 2 Luftfeuchtigkeit relativ (%)
- Klima: 3 Luftfeuchtigkeit absolut (g/m³)
- Klima: 4 Lichtintensität (Lux)
- Klima: 5 Licht-Status (Nacht / Optimal / Intensiv / Lichtstress)
- Klima: 6 Taupunkt (°C)
- Klima: 6b Schimmel-Risiko (Sicher / Warnung / Gefahr)
- Klima: 7 VPD (kPa)
- Klima: 8 VPD Status (Phase → Stecklinge / Wachstum / Blüte)

**Berechnete Werte:**
- **Taupunkt:** Magnus-Formel (ab wann bildet sich Kondenswasser?)
- **VPD:** Tetens-Formel (Sättigungsdefizit → Verdunstungsdruck)
- **Absolute Feuchte:** Berechnung aus Temp. + rel. Feuchte

**Schwellenwerte:**
```yaml
# Licht (in Lux)
light_night: "15"           # Nacht / Dunkelphase
light_seed: "15000"         # Sämlinge / Stecklinge
light_veg: "40000"          # Wachstumsphase (Vegetativ)
light_flower: "75000"       # Blütephase

# VPD (in kPa)
vpd_low: "0.6"              # Zu niedrig
vpd_ideal_seed: "0.8"       # Stecklinge & Keimung
vpd_ideal_veg_early: "0.9"  # Frühe Wachstumsphase
vpd_ideal_veg_late: "1.1"   # Späte Wachstumsphase
vpd_ideal_flower: "1.6"     # Blütephase
```

### 4.3 Modul: ENERGIE (`energie.yaml`)
**Messungen:**
- Energie: 1 Akku Zustand (Text: Kritisch / Bald nachladen / Gut / Vollständig / Ladegerät angeschlossen)
- Energie: 2 Akku Ladestand (%)
- Energie: 3 Akku Spannung (V)

**Wie es funktioniert:**
- ADC auf Pin 33 misst die Zellspannung (über Spannungsteiler ÷ 2)
- Kalibrierung: 3,51 V → 0% | 4,20 V → 100%
- > 4,3 V = Ladegerät angesteckt (`is_powered = true`)
- Unter 4,3 V = Akkubetrieb (`is_powered = false`)
- `current_temp_offset` wird dynamisch angepasst (Abwärme-Kompensation)

### 4.4 Modul: SYSTEM (`system.yaml`)
**Messungen:**
- System: 01 ESPHome Version
- System: 02 Neustart-Grund (Power-Cycle / Wake / etc.)
- System: 03 Speicher Auslastung (RAM in Bytes)
- System: 04 Hostname
- System: 05 MAC-Adresse
- System: 06 IP-Adresse
- System: 07 WLAN SSID
- System: 08 WLAN BSSID (MAC des Access Points)
- System: 08b WLAN AP Name (gemappt via secrets.yaml)
- System: 09 WLAN Signalstärke (dBm / Prozent)
- System: 10 Betriebszeit (Sekunden seit letztem Boot)
- System: 11 Letztes Update (Uhrzeit aus SNTP)

**Schalter:**
- Deep Sleep blockieren (für OTA-Updates)
- Kalibrierungsmodus (zeigt Rohwerte in der Konsole)
- Stromversorgung Sensoren (Ein-/Ausschalten)

**Deep Sleep Logik:**
Das Board schaltet sich automatisch nach 15 Minuten aus, wenn:
- ✓ MQTT verbunden
- ✓ Deep Sleep nicht blockiert
- ✓ Kalibrierungsmodus aus
- ✓ Im Akkubetrieb (nicht am Strom)
- ✓ Temperatur-Sensor hat gültigen Wert

*Fallback:* Wenn MQTT 60 Sekunden offline ist → Force Deep Sleep!

### 4.5 Modul: ERWEITERT (`erweitert.yaml`)
**Messungen:**
- Klima: 5b Photoperiode (Lichtphase / Dunkelphase)
- Klima: 6b Schimmel-Risiko (Sicher / Warnung / Erhöhtes Risiko / Akute Gefahr)
- Klima: 9 DLI Schätzwert (mol/m²/d → Tagesakkumulation)
- Klima: 9b Hitzestress-Index (Akkumulierte Minuten > 30°C)
- System: 09 WLAN Signalqualität (in Prozent)

**DLI (Daily Light Integral):**
- Misst die tägliche Lichtenergiesumme in mol/m²/d
- Umrechnung: Lux → PPFD → Akkumulation pro Minute
- Setzt sich um 00:00 Uhr automatisch zurück
- Typische Zielwerte: 12–14 mol/m²/d für Blüte

**Hitzestress-Index:**
- Zählt jede Minute, in der Temperatur > 30°C
- Setzt sich um 00:00 Uhr automatisch zurück
- Zeigt Belastung durch Überwärmung an

---

## 5. Sensoren verstehen

### 5.1 Bodenfeuchte-Sensor (kapazitiv)
**Funktionsweise:**
Der kapazitive Sensor misst die Dielektrizitätskonstante (Permittivität) des Bodens. Wasser hat eine hohe Permittivität → hohe Kapazität → höhere Spannungsmessung

**Vorteile:**
- ✓ Keine Korrosion durch Strom (weniger Drift als resistive Sensoren)
- ✓ Schnelle Messung
- ✓ Robust gegen Verdichtung

**Nachteile:**
- ✗ Braucht individuelle Kalibrierung pro Bodentyp
- ✗ Kann durch Salzgehalt beeinflusst werden
- ✗ Temperaturabhängig (wird durch das System kompensiert)

### 5.2 EC-Sensor (Elektrische Leitfähigkeit)
**Funktionsweise:**
Der EC-Sensor misst die Fähigkeit des Bodens, Elektrizität zu leiten. Gelöste Salze (Nährstoffe) erhöhen die Leitfähigkeit

**Einheit:** mS/cm (Millisiemens pro Zentimeter)
- 0,0–0,3 mS/cm: Sehr arm an Nährstoffen (Regenwasser, Anzucht)
- 0,3–0,8 mS/cm: Nährstoffarm (Torferde, Sämlinge)
- 0,8–1,5 mS/cm: OPTIMAL (normal gedüngte Gartenerde)
- 1,5–2,2 mS/cm: Erhöhter Salzgehalt (Vorsicht vor Überdüngung!)
- \> 2,2 mS/cm: ÜBERDÜNGT (Wurzeln in Gefahr!)

⚠️ **Wichtig: EC ändert sich mit der Bodenfeuchte!**
- Trockener Boden → EC-Wert steigt (Konzentration höher)
- Nasser Boden → EC-Wert fällt (Verdünnung)

### 5.3 DHT11 Temperatur- & Feuchte-Sensor
**Spezifikationen:**
- Temperaturbereich: 0–50°C (genau: ±2°C)
- Feuchtebereich: 20–90% (genau: ±5%)
- Messintervall: min. 2 Sekunden
- Genauigkeit: ±1°C, ±1% RH

**Temperatur-Offset:**
Der DHT11 ist auf der Platine angebracht und erwärmt sich durch die elektronischen Komponenten.
Das Projekt kompensiert das automatisch:
- Im Akkubetrieb: -0,1°C Offset
- Bei Stromversorgung: -4,5°C Offset (mehr Eigenabwärme)

### 5.4 BH1750 Lichtsensor
**Spezifikationen:**
- Messbereich: 1–65535 Lux
- Kommunikation: I2C (0x23)
- Spektralempfindlichkeit: ähnlich dem menschlichen Auge

**Lux-Werte für verschiedene Situationen:**
- 1 Lux: Sternenlicht, völlige Finsternis
- 50 Lux: Beleuchteter Raum (Nachts)
- 500 Lux: Büro, bewölkter Tag
- 5000 Lux: Sonniger Innenraum (am Fenster)
- 10000+ Lux: Direktes Sonnenlicht draußen
- 50000+ Lux: Sehr intensive Beleuchtung (LED-Grow-Licht)

⚠️ **Hinweis: BH1750 Multiplikator**
Der Standard-Multiplikator beträgt 1,0. Im Projekt ist er auf 50,0 eingestellt.
Das liegt daran, dass manche BH1750-Module eine unterschiedliche Empfindlichkeit haben. Falls deine Werte nicht stimmen: Diesen Wert in `t-higrow-main.yaml` anpassen (`bh1750_multiplikator`).

### 5.5 Akkumulator (18650 Lithium-Zelle)
**Typische Spannungen während des Betriebs:**
- 4,20 V: Vollständig geladen
- 4,00 V: 80% Kapazität
- 3,70 V: 50% Kapazität (optimal für Lagerung)
- 3,60 V: Warnung (bald nachladen)
- 3,51 V: Kritisch (Sensoren zeigen ungenaue Werte)
- < 3,51 V: Board schaltet möglicherweise ab

**Deep-Sleep Stromverbrauch:**
- Durchschnittlicher Deep Sleep: ~100–200 µA
- 18650 Kapazität: ~2500–3000 mAh
- Theoretische Laufzeit: ~1–2 Wochen (abhängig von Sensor-Aktivität)

---

## 6. Kalibrierung Schritt-für-Schritt

### 6.1 Vor der Kalibrierung
⚠️ **WICHTIG: Die Kalibrierung ist notwendig, um genaue Messwerte zu erhalten!**
- Jede Bodenart (Sand, Lehmerde, Torferde, Coco-Mix) braucht unterschiedliche Kalibrierwerte
- Auch unterschiedliche Sensor-Chargen können variieren
- Eine genaue Kalibrierung ist die Grundlage für korrektes Pflanzenbau-Management!

### 6.2 Bodenfeuchte-Kalibrierung
**Schritt 1: Kalibrierungsmodus aktivieren**
→ Home Assistant → Schalter → "System: Kalibrierungsmodus" → EIN
→ Der Sensor gibt jetzt Rohwerte statt Prozente aus

**Schritt 2: Trocken-Wert messen**
1. Hol einen trockenen Topf mit trockener Erde heraus
2. Steck den Sensor vollständig in die trockene Erde ein
3. Öffne die Web-Konsole: `http://<IP>:80`
4. Schau auf "Boden: 1 Feuchte" - Du siehst jetzt einen Rohwert (z.B. "2400")
5. Nimm den Median von 5 Messungen (um Rauschen zu minimieren)
6. Notiere diesen Wert als `soil_moist_raw_dry`

**Schritt 3: Nass-Wert messen**
1. Nimm einen Behälter mit Wasser (Regenwasser oder destilliert ist ideal)
2. Steck den Sensor vollständig ins Wasser ein
3. Warte 10 Sekunden (Stabilisierung)
4. Notiere wieder den Rohwert (z.B. "1200")
5. Das ist `soil_moist_raw_wet`

**Schritt 4: Werte in secrets.yaml eintragen**
```yaml
soil_moist_raw_dry: "2400"
soil_moist_raw_wet: "1200"
```

**Schritt 5: ESPHome flashen**
```bash
esphome run t-higrow-main.yaml
```
*(Das Board wird mit den neuen Werten neu programmiert)*

### 6.3 EC-Kalibrierung (5-Punkt)
Die EC-Kalibrierung ist genauer und zuverlässiger mit 5 Punkten:

**Benötigte Lösungen:**
- P0: Destilliertes Wasser oder Regenwasser (0,0 mS/cm)
- P1: Nasse Torferde ohne Düngung (ca. 0,3 mS/cm)
- P2: Nasse Anzuchterde (ca. 0,8 mS/cm)
- P3: Nasse Gartenerde, normal gedüngt (ca. 1,5 mS/cm)
- P4: Gesättigte Salzlösung (ca. 5,0 mS/cm)

**Messablauf:**
1. Kalibrierungsmodus AN
2. Sensor in P0 (Wasser) stecken → Rohwert notieren (Median von 5)
3. Sensor abspülen, trocknen, in P1 stecken → Rohwert notieren
4. Wiederhole für P2, P3, P4
5. Alle Werte in `secrets.yaml` eintragen

**Beispiel:**
```yaml
soil_ec_raw_p0: "50"      soil_ec_target_p0: "0.0"
soil_ec_raw_p1: "200"     soil_ec_target_p1: "0.3"
soil_ec_raw_p2: "500"     soil_ec_target_p2: "0.8"
soil_ec_raw_p3: "900"     soil_ec_target_p3: "1.5"
soil_ec_raw_p4: "3000"    soil_ec_target_p4: "5.0"
```

### 6.4 Tipps für genaue Kalibrierung
- ✓ Verwende immer die gleiche Bodenart für die Messung, die später im Topf sein wird
- ✓ Lass den Sensor nach jeder Messung kurz stabilisieren (5–10 Sekunden)
- ✓ Nimm Mehrfachmessungen und bilde den Median
- ✓ Die Kalibrierung sollte bei gleicher Temperatur stattfinden wie der spätere Betrieb
- ✓ Notiere die Temperatur bei der Messung (für Nachverfolgung)
- ✓ Wiederhole die Kalibrierung, wenn du die Bodenart wechselst

---

## 7. VPD & Klima-Konzepte

### 7.1 Was ist VPD?
**VPD = Vapor Pressure Deficit (Sättigungsdefizit)**
Das ist die Differenz zwischen dem Wasserdampfdruck, den die Luft bei einer gegebenen Temperatur aufnehmen könnte, und dem tatsächlichen Wasserdampfdruck.
Vereinfacht: Je höher der VPD, desto trockener die Luft → desto mehr transpirieren die Pflanzen.

### 7.2 Ideale VPD-Werte nach Wachstumsphase
- **VPD < 0,6 kPa:** ⚠️ Zu niedrig = Pflanze transpiriert nicht optimal (Risiko: Pilzkrankheiten, Staunässe-Probleme)
- **VPD 0,6–0,8 kPa (IDEAL für Sämlinge/Stecklinge):** ✓ Schwache Pflanzen brauchen höhere Luftfeuchte ✓ Reduziert Verdunstungsstress
- **VPD 0,9–1,1 kPa (IDEAL für Vegetatives Wachstum):** ✓ Optimales Wachstum ✓ Gutes Gleichgewicht zwischen Transpiration & Wurzelnährstoff-Aufnahme
- **VPD 1,1–1,6 kPa (IDEAL für Blüte):** ✓ Unterstützt Blütenentwicklung ✓ Höhere Verdunstung fördert Nährstoff-Translokation
- **VPD > 1,6 kPa:** ⚠️ Zu hoch = Trockenstress (Risiko: Blatt-Randnekrosen, reduziertes Wachstum, Blüten-Probleme)

### 7.3 Praktische VPD-Regulierung
**VPD zu niedrig? → Lösungsansätze:**
- ✓ Temperatur erhöhen (VPD steigt exponentiell mit Temp.)
- ✓ Luftzirkulation verbessern (Ventilator)
- ✓ Feuchte leicht reduzieren (nicht brutal!)

**VPD zu hoch? → Lösungsansätze:**
- ✓ Feuchte erhöhen (Luftbefeuchter, Vernebler)
- ✓ Temperatur senken (weniger Heizung/Licht-Abstand)
- ✓ Luftzirkulation reduzieren

### 7.4 DLI (Daily Light Integral)
**DLI** = akkumulierte Lichtmenge pro Tag in mol/m²/Tag

**Typische DLI-Zielwerte:**
- Sämlinge: 3–6 mol/m²/Tag
- Vegetatives Wachstum: 8–12 mol/m²/Tag
- Blüte: 12–18 mol/m²/Tag
- Maximales Wachstum: 20–25 mol/m²/Tag (aber Risiko von Lichtstress)

### 7.5 Taupunkt & Schimmelrisiko
**Taupunkt** = Temperatur, ab der sich Wasser kondensiert
Berechnung: Magnus-Formel (implementiert im Projekt)

**Schimmelrisiko Indikatoren:**
- 🟢 **Sicher:** Temp – Taupunkt > 5°C
- 🟡 **Warnung:** Temp – Taupunkt = 3–5°C UND Feuchte > 60%
- 🟠 **Erhöhtes Risiko:** Temp – Taupunkt = 3°C UND Feuchte > 65%
- 🔴 **AKUTE GEFAHR:** Temp – Taupunkt < 3°C UND Feuchte > 75% (Kondensat bildet sich!)

**Tipps zur Schimmel-Prävention:**
- ✓ Gute Luftzirkulation (Ventilator, aber nicht direkt auf Blätter)
- ✓ Nachts Feuchte nicht zu hoch halten (< 60% RH)
- ✓ Temperaturgradienten zwischen Innen/Außen minimieren
- ✓ Blätter morgens trocknen lassen
- ✓ Dichte Bestände auslichten (bessere Luftzirkulation)

---

## 8. Fehlerbehandlung & Troubleshooting

### 8.1 WLAN-Verbindung funktioniert nicht
**Problem:** Board verbindet sich nicht mit dem WLAN
**Lösungsschritte:**
1. Überprüfe SSID & Passwort in `secrets.yaml` auf Tippfehler
2. Stelle sicher, dass das WLAN 2.4 GHz ist (ESP32 unterstützt kein 5 GHz!)
3. Setze das Board auf Werkseinstellungen zurück: `esphome clean t-higrow-main.yaml`
4. Flashe neu und warte 30 Sekunden
5. Wenn immer noch Probleme: Nutze das Fallback-AP ("Garten-Fallback" mit PW 12345678)

### 8.2 Sensoren zeigen "Fehler" oder NAN
**Problem:** Sensor-Werte sind leer oder zeigen "Fehler"
- **Feuchte-Sensor (NAN):** → ADC Kalibrierung prüfen → Sensor ggfs. austauschen (Hardware-Defekt)
- **DHT11 (Temperatur/Feuchte NAN):** → Sensor könnte beschädigt sein → Zu schnelle Messungen: Update-Intervall auf 3s erhöhen → Stromversorgung prüfen
- **BH1750 (Licht NAN):** → I2C-Adresse falsch? (sollte 0x23 sein) → Sensor abdecken und wieder freigeben (Reset)

### 8.3 Akkuspannung zeigt 0V
**Problem:** "Akku Spannung" zeigt 0,0 V
**Ursachen:**
- ❌ Spannungsteiler-Widerstand beschädigt
- ❌ ADC-Pin 33 defekt
- ❌ Keine Batterie eingelegt

**Überprüfung:**
1. Batterie einlegen & prüfen, ob LED blinkt
2. Batteriespannung mit Multimeter prüfen (sollte 3,7–4,2 V sein)
3. Wenn Multimeter Spannung zeigt, aber Board 0V: Board-Reparatur nötig

### 8.4 Deep Sleep aktiviert sich nicht
**Problem:** Board geht nicht in Sleep, verbraucht Batterie
**Gründe:**
- ✓ Deep Sleep ist blockiert (Schalter ist AN)
- ✓ Kalibrierungsmodus ist aktiv
- ✓ Stromversorgung (USB) ist angesteckt
- ✓ MQTT verbindung fehlgeschlagen (wartet auf Fallback: 60s)

**Überprüfung:**
1. Home Assistant → Integrations → ESPHome → Geräte
2. Prüfe den Schalter "Deep Sleep blockieren" → sollte AUS sein
3. Kalibrierungsmodus prüfen → sollte AUS sein
4. USB-Kabel abziehen
5. MQTT-Broker prüfen (sollte online sein)

### 8.5 Bodenfeuchte-Wert springt wild hin und her
**Problem:** Messwerte sind instabil
**Ursachen & Lösungen:**
- Sensor sitzt nicht tief genug → tiefer einführen
- Kalibrierung falsch → Re-Kalibrierung durchführen
- Bodenart hat sich verändert → Neue Kalibrierung!
- Filter-Einstellungen anpassen (`median_window` erhöhen auf 7–10)
- Strom-Versorgung: Battery liefert zu wenig Strom → USB nutzen?

---

## 9. Häufig Gestellte Fragen (FAQ)

**F: Kann ich mehrere Boards gleichzeitig betreiben?**
A: Ja! Das Projekt nutzt MAC-Suffixe für eindeutige Gerätenamen. Jedes Board bekommt automatisch eine andere IP. Nutze das Pflanzenmapping in der `secrets.yaml`, um ihnen Namen zu geben.

**F: Wie lange hält die 18650-Batterie im Deep Sleep?**
A: Mit 15-Minuten-Intervallen: ca. 10–14 Tage (hängt von Batterie-Kapazität ab). Eine 3000 mAh Zelle + typischer 100 µA Deep-Sleep-Strom = sehr lange Laufzeit!

**F: Brauche ich einen MQTT-Broker?**
A: Nein, aber es ist stark empfohlen. Du kannst auch nur die ESPHome API verwenden (dann brauchst du Home Assistant). MQTT ist optional, macht aber Home Assistant Integration stabiler.

**F: Kann ich die Deep-Sleep-Zeit anpassen?**
A: Ja! In `t-higrow-main.yaml` änderst du: `deep_sleep_duration: "15min"` auf z.B. `"30min"` oder `"5min"`. ESPHome flashen und fertig.

**F: Wie verbinde ich das Board mit Home Assistant?**
A: Automatisch! Nach dem Flashen sollte das Board in Home Assistant → Einstellungen → Integrations → ESPHome auftauchen. Einmal klicken zum Hinzufügen.

**F: Kann ich die Sensoren einzeln ein-/ausschalten?**
A: Ja! Das Projekt hat eine modulare Struktur (packages). In `t-higrow-main.yaml` kannst du einzelne Module auskommentieren (mit `#`).

**F: Was ist der Unterschied zwischen Boden-Feuchte und EC?**
A: Bodenfeuchte misst Wasser %, EC misst Salzgehalt (Nährstoffe). Beides zusammen zeigen die Nährstoff-Verfügbarkeit. Hohe Feuchte + niedriges EC = trockene Nährstoffe möglich.

**F: Kann das Board außerhalb beschädigt werden?**
A: Das Board ist wasserresistent (Sensoren sind on-board), aber nicht vollständig wasserdicht. Halte es trocken. Die Bodenfeuchte- und EC-Sensoren sollten trocken gewischt werden nach der Messung.

**F: Wie debugge ich Fehler?**
A: Nutze die Web-Konsole (`http://<IP>:80`) oder: `esphome logs t-higrow-main.yaml`. Im Kalibrierungsmodus siehst du Rohwerte für alle Sensoren.

---

**Fertig! 🎉**  
Du hast die vollständige Dokumentation durchgearbeitet. Viel Erfolg mit deinem Cannabis Monitor!  
**GitHub:** https://github.com/ClubMenthol/T-Higrow
