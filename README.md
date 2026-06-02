# T-Higrow Cannabis Monitor (v4.7.0)

Ein modularer, energieoptimierter IoT-Sensor für die Überwachung von Boden- und Klimawerten beim Cannabis-Anbau (Garten/Indoor). Das Projekt basiert auf dem **LILYGO T-Higrow ESP32 (V1.1 / DHT11)** Board und nutzt **ESPHome** für eine nahtlose Integration in Home Assistant oder via MQTT.


---

## 💡 Hauptmerkmale (Features)

* **Vollständige Modularität (v4.7.0):** Die Konfiguration ist in saubere Sub-Pakete unterteilt (`system`, `klima`, `boden`, `energie`), die sich im Hauptskript flexibel aktivieren oder deaktivieren lassen.
* **Intelligentes Energiemanagement:** Deep-Sleep-Unterstützung (Standard: 15 Minuten) mit dynamischer Deaktivierung, sobald ein Ladegerät angeschlossen ist, das System im Kalibrierungsmodus läuft oder ein OTA-Update blockiert wird.
* **Präzise Sensor-Filterung:** Nutzung von ESPHome-Medianfiltern für alle kritischen Analog- und Klimawerte, um Messfehler und Signalrauschen effektiv zu eliminieren.
* **Erweiterte Klimaberechnungen:** Automatische Ermittlung von **Taupunkt**, **absoluter Luftfeuchtigkeit** und dem **Dampfdruckdefizit (VPD)** inklusive einer dynamischen Text-Statusausgabe für die jeweilige Pflanzenphase.
* **2-Wege-Kalibrierung:** Komfortabler Kalibrierungsmodus über die Weboberfläche, um Rohwerte (Volt) für Bodenfeuchte und die elektrische Leitfähigkeit (EC / Düngergehalt) direkt im Live-Log einzusehen.
* **Dual-Betriebsmodus:** Automatische Temperaturoffset-Anpassung je nachdem, ob das Gerät per Akku (`-0.1°C`) oder per Dauerstrom (`-4.5°C`) betrieben wird, um die Eigenerwärmung des Boards zu kompensieren.

---

## 📂 Repository-Struktur

Dein Projektverzeichnis sollte wie folgt aufgebaut sein:

```text
├── t-higrow-main.yaml     # Hauptskript (Zentrale Variablen & Hardware-Setup)
├── secrets.yaml           # Deine privaten Netzwerk- & MQTT-Zugangsdaten (lokal)
├── secrets.yaml-beispiel  # Vorlage für secrets.yaml (wird auf GitHub hochgeladen)
└── modules/               # Unterverzeichnis für die modularisierten Pakete
    ├── system.yaml        # Deep Sleep, System-Sensoren (WLAN, RAM, Uptime)
    ├── klima.yaml         # DHT11, BH1750, VPD- & Taupunktberechnung
    ├── boden.yaml         # Kapazitive Bodenfeuchte & EC-Kennlinie
    └── energie.yaml       # ADC-Akkumessung, Prozentberechnung & Ladestatus

🛠️ Schnellstart & Dokumentation
Für dieses Projekt stehen detaillierte Anleitungen zur Verfügung:

HOWTO.md (Installations- & Einrichtungsanleitung): Erfährst du Schritt für Schritt, wie du die Firmware installierst, deine secrets.yaml konfigurierst und die Sensoren exakt kalibrierst.

MANUAL.md (Bedienungsanleitung & technische Details): Erklärt die Funktionsweise der Schalter, die Logik hinter den VPD-/EC-Grenzwerten und das Verhalten des Deep-Sleep-Modus.

📋 Voraussetzungen (Hardware)
Board: LILYGO T-Higrow ESP32 (Version 1.1 mit On-Board DHT11)

Zusatz-Sensor (optional): BH1750 I2C Lichtsensor (an den Pins SDA:25 / SCL:26)

Stromversorgung: 18650 Lithium-Ionen-Zelle oder USB-C Netzteil

⚖️ Lizenz & GitHub
Dieses Projekt ist für die private Nutzung und Optimierung deines Gartens gedacht. Das Teilen und Erweitern der Codebasis ist ausdrücklich erwünscht!
