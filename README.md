# 🌱 T-Higrow Cannabis Monitor

Ein modularer ESPHome-Multisensor auf Basis des **LILYGO T-Higrow ESP32 Boards** zur präzisen Erfassung und Steuerung von Boden-, Licht- und Klimaparametern für den Pflanzenbau.

[![ESPHome Version](https://img.shields.io/badge/ESPHome-v4.9.6-brightgreen.svg)](https://esphome.io/)
[![Hardware](https://img.shields.io/badge/Hardware-LILYGO_T--Higrow_ESP32-blue.svg)](https://github.com/ClubMenthol/T-Higrow)
[![License](https://img.shields.io/badge/License-MIT-orange.svg)](LICENSE)

---

## 📋 Überblick & Features

- **Boden-Monitoring:** Kapazitive Feuchtemessung (0–100 %) & 5-Punkt-kalibrierte Leitfähigkeit/EC (0–5 mS/cm).
- **Klima-Tracking:** Temperatur, rel./abs. Luftfeuchte (DHT11), Taupunkt- & Schimmel-Risiko-Erkennung.
- **Licht & DLI:** Beleuchtungsstärke (BH1750, Lux), Photoperioden-Tracking und tägliche DLI-Akkumulation ($mol/m^2/d$).
- **VPD-Berechnung:** Automatischer Phasen-Status (Stecklinge, Wachstum, Blüte) nach Tetens-Formel.
- **Batteriemanagement:** Automatische Akkustandüberwachung (18650 Zelle), Abwärme-Kompensation & intelligenter Deep Sleep (15 Min.).
- **Integration:** Native Home Assistant Anbindung (API) & MQTT mit Offline-Fallback.

---

## ⚡ Schnellstart

### 1. Repository klonen & vorbereiten
```bash
git clone https://github.com/ClubMenthol/T-Higrow.git
cd T-Higrow
```

### 2. Konfiguration anlegen (`secrets.yaml`)
Erstelle im Hauptverzeichnis eine `secrets.yaml` mit deinen Zugangs- und Kalibrierungsdaten:

```yaml
devicename: "t-higrow-001"
friendly_name: "Pflanze 1"

wifi_ssid: "DeinWLAN"
wifi_password: "DeinPasswort"

manual_static_ip: "192.168.178.100"
manual_gateway: "192.168.178.1"
manual_subnet: "255.255.255.0"
manual_dns1: "1.1.1.1"

api_encryption_key: "GENERIERTER_SCHLUESSEL" # Erzeugen mit: esphome generate-key
mqtt_broker: "192.168.178.50"

# Kalibrierungswerte (Rohwerte)
soil_moist_raw_dry: "2400"
soil_moist_raw_wet: "1200"

# EC 5-Punkt Kalibrierung (Rohwerte)
soil_ec_raw_p0: "0"       soil_ec_target_p0: "0.0"
soil_ec_raw_p1: "200"     soil_ec_target_p1: "0.3"
soil_ec_raw_p2: "500"     soil_ec_target_p2: "0.8"
soil_ec_raw_p3: "900"     soil_ec_target_p3: "1.5"
soil_ec_raw_p4: "3000"    soil_ec_target_p4: "5.0"
```

### 3. Flashen & Überwachen
```bash
esphome run t-higrow-main.yaml
```

Logs live einsehen:
```bash
esphome logs t-higrow-main.yaml
```

---

## 🛠 Modulstruktur

```text
esphome/
├── t-higrow-main.yaml         # Hauptkonfiguration & Schwellenwerte
├── secrets.yaml               # Zugangsdaten & Sensor-Offsets (privat)
└── modules/
    ├── system.yaml            # WLAN, Deep Sleep, Diagnose & OTA
    ├── klima.yaml             # DHT11, BH1750, Taupunkt & VPD-Logik
    ├── boden.yaml             # Feuchte & 5-Punkt EC-Interpolation
    ├── energie.yaml           # 18650 Akkustatus & Stromversorgung
    ├── erweitert.yaml         # DLI-Akkumulation & Hitzestress-Tracker
    └── pflanzen_mapping.yaml  # MAC-basierte Namenszuweisung
```

---

## 🧪 Kalibrierung (Kurzanleitung)

1. **Kalibrierungsmodus aktivieren:** In Home Assistant den Schalter `System: Kalibrierungsmodus` aktivieren (gibt Rohwerte auf ADC aus).
2. **Bodenfeuchte:** 
   - Sensor trocken an Luft/Erde messen $
ightarrow$ `soil_moist_raw_dry`
   - Sensor in Wasser tauchen $
ightarrow$ `soil_moist_raw_wet`
3. **EC-Sensor (5 Punkte):**
   - Rohwerte in Testlösungen messen: P0 (0.0 mS), P1 (0.3 mS), P2 (0.8 mS), P3 (1.5 mS), P4 (5.0 mS).
4. **Werte eintragen:** In `secrets.yaml` übertragen und Firmware neu flashen.

---

## 🌿 VPD Zielwerte

| Phase | Optimaler VPD | Zielbereich |
| :--- | :--- | :--- |
| **Sämlinge / Stecklinge** | 0.6 – 0.8 kPa | Hohe Feuchte, geringer Verdunstungsdruck |
| **Vegetativ (Wachstum)** | 0.9 – 1.1 kPa | Ausgewogene Nährstoffaufnahme |
| **Blütephase** | 1.1 – 1.6 kPa | Optimierte Transpiration |
| **Kritisch** | < 0.6 oder > 1.6 kPa | Schimmelrisiko bzw. Trockenstress |

---

## 🔗 Weiterführende Links

- Ausführliche Anleitung: [MANUAL.md](./MANUAL.md)
- GitHub Repository: [https://github.com/ClubMenthol/T-Higrow](https://github.com/ClubMenthol/T-Higrow)
