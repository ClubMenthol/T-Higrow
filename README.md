# T-Higrow Cannabis Monitor

Ein modularer, energieoptimierter IoT-Sensor für die Überwachung von Boden- und Klimawerten beim Cannabis-Anbau (Garten/Indoor). Das Projekt basiert auf dem **LILYGO T-Higrow ESP32 (V1.1 / DHT11)** Board und nutzt **ESPHome** für eine nahtlose Integration in Home Assistant oder via MQTT.

## 💡 Hauptmerkmale (Features)

* **Vollständige Modularität:** Die Konfiguration ist in saubere Sub-Pakete unterteilt (`system`, `klima`, `boden`, `energie`), die sich im Hauptskript flexibel aktivieren oder deaktivieren lassen.
* **Intelligentes Energiemanagement:** Deep-Sleep-Unterstützung (Standard: 15 Minuten) mit dynamischer Deaktivierung, sobald ein Ladegerät angeschlossen ist, das System im Kalibrierungsmodus läuft oder ein OTA-Update blockiert wird.
* **Präzise Sensor-Filterung:** Nutzung von ESPHome-Medianfiltern für alle kritischen Analog- und Klimawerte, um Messfehler und Signalrauschen effektiv zu eliminieren.
* **Erweiterte Klimaberechnungen:** Automatische Ermittlung von **Taupunkt**, **absoluter Luftfeuchtigkeit** und dem **Dampfdruckdefizit (VPD)** inklusive einer dynamischen Text-Statusausgabe für die jeweilige Pflanzenphase.
* **2-Wege-Kalibrierung:** Komfortabler Kalibrierungsmodus über die Weboberfläche, um Rohwerte (Volt) für Bodenfeuchte und die elektrische Leitfähigkeit (EC / Düngergehalt) direkt im Live-Log einzusehen.
* **Dual-Betriebsmodus:** Automatische Temperaturoffset-Anpassung je nachdem, ob das Gerät per Akku (`-0.1°C`) oder per Dauerstrom (`-4.5°C`) betrieben wird, um die Eigenerwärmung des Boards zu kompensieren.

<<<<<<< HEAD
So sieht das Webinterface aus.
![Screenshot.](https://github.com/ClubMenthol/T-Higrow/blob/main/screencapture-t-higrow.png)

Der Kalibrierungsmodus ist aktiv. Hier könne die Rohwerte abgelesen werden, die zur Kalibrierung benötigt werden.
=======
🔌 Hardware-Spezifikationen

* Controller: LILYGO T-Higrow ESP32 (Version 1.1 mit On-Board DHT11)
* Klimasensor: DHT11 (On-Board, Pin 16)
* Lichtsensor: BH1750 (I2C: SDA=Pin 25, SCL=Pin 26, Adresse 0x23)
* Bodenfeuchte: Kapazitiver Sensor (Analog-Eingang Pin 32)
* Boden-EC: Leitfähigkeitssensor (Analog-Eingang Pin 34)
* Batterie-Messung: Interner Spannungsteiler (Analog-Eingang Pin 33, Multiplikator 2.0)
* Sensor-Power-Gate: GPIO Pin 4 (Schaltet die Sensoren zur Stromeinsparung ab)
* Wake-Taster: Physischer Button an Pin 35 (Invertiert, weckt das Board manuell)
* Stromversorgung: 18650 Lithium-Ionen-Zelle oder USB-C Netzteil

🛠️ Installation & Einrichtung

1. Vorbereitung der Secrets
Kopiere die Datei secrets-beispiel.yaml und benenne sie in secrets.yaml um.
Passe dort deine persönlichen Netzwerkdaten, Passwörter und statischen IP-Adressen an.

2. Kompilieren und Flashen
Verbinde dein LILYGO T-Higrow per USB-C mit deinem Computer oder ESPHome-Server.
Nutze den folgenden Terminal-Befehl, um das Projekt zu validieren, zu kompilieren und direkt auf das Board zu schreiben:

Bashesphome run t-higrow-main.yaml

Tipp: Sobald das Board einmal im WLAN ist, können zukünftige Updates komfortabel kabellos via OTA (Over-The-Air) eingespielt werden.
Aktiviere dazu im Home Assistant Dashboard einfach den Schalter "System: Wach bleiben", um den Deep Sleep temporär zu unterdrücken.

📐 Kalibrierungsanleitung

Für präzise Messergebnisse müssen die analogen Rohwerte der Bodensensoren einmalig kalibriert werden.
1. Schalte in Home Assistant den Schalter "System: Kalibrierungsmodus (Rohwerte)" ein.
2. Beobachte die ESPHome-Webkonsole oder das Home Assistant Log. Das System gibt nun alle 2 Sekunden die ungefilterten Volt-Rohwerte aus.
>>>>>>> 9d1a57b (Update der ESPHome-Module, Anpassung der Dokumentation und Refactoring der Secrets-Beispieldatei)

![Screenshot.](https://github.com/ClubMenthol/T-Higrow/blob/main/kalibrierung.png)
---

Bodenfeuchte kalibrieren (soil_moist_raw)
* Trockenwert (0%): Lass den Sensor komplett trocken an der Luft liegen. Notiere dir den ausgegebenen Volt-Wert (z. B. 2.471) und trage ihn in deiner secrets.yaml bei soil_moist_raw_dry ein.
* Nasswert (100%): Tauche den Sensor bis zur Markierung in ein Glas Wasser. Notiere den Volt-Wert (z. B. 1.372) und trage ihn bei soil_moist_raw_wet ein.

Dünger / EC-Wert kalibrieren (soil_ec_raw)
Der EC-Sensor nutzt eine präzise 5-Punkt-Leistungskurve (p0 bis p4), um Spannungen in $mS/cm$ zu übersetzen. Die Standard-Zielwerte (target) sind im Skript vordefiniert. Ermittle die Rohwerte wie folgt und trage sie in die secrets.yaml ein:
* p0 (0.0 mS/cm): Sensor komplett trocken oder in reinem Regenwasser/destilliertem Wasser.
* p1 (0.3 mS/cm): Sensor in nasser, sehr nährstoffarmer Torferde bzw. Anzuchterde.
* p2 (0.8 mS/cm): Sensor in leicht gedüngter, feuchter Erde.
* p3 (1.5 mS/cm): Sensor in optimal versorgter, feuchter Gartenerde.
* p4 (5.0 mS/cm): Sensor in einer gesättigten Salzlösung (Maximalwert).

🔄 Dashboard-Import & Updates
Dank des integrierten dashboard_import-Blocks ist dieses Projekt vollständig updatefähig. Wenn das Skript auf GitHub liegt, kann es vom ESPHome Dashboard direkt getrackt werden:

dashboard_import:
  package_import_url: github://ClubMenthol/T-Higrow/t-higrow-main.yaml@main
  import_full_config: false
  
Fügst du ein neues Board hinzu, musst du lediglich dessen MAC-Suffix und den Wunschnamen am Anfang der t-higrow-main.yaml (oder in den Secrets) erweitern:

mac_p3: "a806a5"
name_p3: "Deine Neue Sorte"

Der C++ Core erledigt den Rest beim nächsten Bootvorgang vollautomatisch!

🛠️ Schnellstart & Dokumentation
Für dieses Projekt stehen detaillierte Anleitungen zur Verfügung:

[HOWTO.md](HOWTO.md) (Installations- & Einrichtungsanleitung): Erfährst du Schritt für Schritt, wie du die Firmware installierst, deine secrets.yaml konfigurierst und die Sensoren exakt kalibrierst.

[MANUAL.md](MANUAL.md) (Bedienungsanleitung & technische Details): Erklärt die Funktionsweise der Schalter, die Logik hinter den VPD-/EC-Grenzwerten und das Verhalten des Deep-Sleep-Modus.
<<<<<<< HEAD

📋 Voraussetzungen (Hardware)
Board: LILYGO T-Higrow ESP32 (Version 1.1 mit On-Board DHT11)

Zusatz-Sensor (optional): BH1750 I2C Lichtsensor (an den Pins SDA:25 / SCL:26)

Stromversorgung: 18650 Lithium-Ionen-Zelle oder USB-C Netzteil
=======
>>>>>>> 9d1a57b (Update der ESPHome-Module, Anpassung der Dokumentation und Refactoring der Secrets-Beispieldatei)

⚖️ Lizenz & GitHub
Dieses Projekt ist für die private Nutzung und Optimierung deines Gartens gedacht. Das Teilen und Erweitern der Codebasis ist ausdrücklich erwünscht!
