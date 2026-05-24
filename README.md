# ESPHome LILYGO T-Higrow Cannabis Monitor

Dieses Repository enthält eine hochentwickelte und optimierte ESPHome-Konfiguration für den **LILYGO T-Higrow ESP32 (V1.1 / DHT11)** zur automatisierten Überwachung von Boden- und Klimawerten in der Pflanzenzucht.

## Features
* **Bodenüberwachung:** Kapazitive Feuchtigkeitsmessung und Leitfähigkeit (EC-Wert) mit integrierter Multi-Punkt-Kalibrierung.
* **Klimaanalyse:** Messung von Temperatur, relativer Luftfeuchtigkeit und Lichtintensität (BH1750).
* **Erweiterte Berechnungen:** Echtzeitberechnung von absolutem Feuchtigkeitsgehalt, Taupunkt und **VPD (Dampfdruckdefizit)** zur optimalen Vitalitätskontrolle.
* **Energieeffizient:** Ausgeklügelte Deep-Sleep-Schleife für autarken Akkubetrieb (18650 Zelle) im Außenbereich.
* **Wartungsmodus:** Integrierter Schalter zur temporären Deaktivierung des Deep Sleep (für problemlose OTA-Updates) sowie ein Live-Kalibrierungsmodus.

## Dateistruktur
* `t-higrow-final.yaml` - Die ESPHome-Hauptkonfigurationsdatei.
* `secrets.yaml` (**Nicht im Repository enthalten!**) - Enthält deine lokalen Netzwerk- und Zugangsdaten. Eine Vorlage befindet sich im Skriptcode.

## Installation & Verwendung
1. Erstelle lokal eine `secrets.yaml` basierend auf den im Skript benötigten Variablen.
2. Passe deine Schwellenwerte und Hardware-Kalibrierungsdaten direkt im Kopfbereich (Modul 1) der `t-higrow-final.yaml` an.
3. Kompiliere und flashe das Projekt via ESPHome-Dashboard oder CLI.

## Lizenz
Dieses Projekt ist unter der MIT-Lizenz lizenziert – siehe die [LICENSE](LICENSE) Datei für Details.
