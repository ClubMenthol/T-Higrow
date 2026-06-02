# HOWTO — Installations- und Kalibrierungsanleitung (v4.7.0)

Diese Anleitung führt dich durch die Ersteinrichtung und die präzise Kalibrierung der analogen Bodensensoren deines Garten-Monitors.

---

## 🛠️ Schritt 1: Dateistruktur vorbereiten

1. Kopiere alle Dateien aus diesem Repository in deinen ESPHome-Ordner (z.B. in Home Assistant im Verzeichnis `/config/esphome/`).
2. Erstelle im Ordner `modules/` die vier Unterdateien (`system.yaml`, `klima.yaml`, `boden.yaml`, `energie.yaml`) und füge den jeweiligen Code ein.

---

## 🔐 Schritt 2: Geheimnisse konfigurieren (`secrets.yaml`)

Erstelle eine Datei namens `secrets.yaml` im Hauptverzeichnis deines ESPHome-Ordners (nutze die `secrets.yaml-beispiel` als Vorlage) und passe deine Zugangsdaten an:

```yaml
wifi_ssid: "DEIN_WLAN_NAME"
wifi_password: "DEIN_WLAN_PASSWORT"
wifi_bssid: "AA:BB:CC:DD:EE:FF"        # BSSID des stärksten APs für schnellen Connect
manual_static_ip: "192.168.178.X"     # Feste IP für schnellen Verbindungsaufbau
manual_gateway: "192.168.178.1"
manual_subnet: "255.255.255.0"
manual_dns1: "192.168.178.1"
manual_dns2: "1.1.1.1"
api_encryption_key: "DEIN_GENERIERTER_OPENSSL_KEY"
mqtt_broker: "192.168.178.XX"          # IP deines MQTT-Brokers / Home Assistant
🚀 Schritt 3: Firmware flashen
Verbinde das T-Higrow-Board per USB-C mit deinem Computer. Nutze das ESPHome-Dashboard oder das Terminal, um das Skript zu kompilieren und zu installieren.

Befehl für das Terminal:
esphome run t-higrow-main.yaml

Hinweis: Beim ersten Booten wartet das Board standardmäßig 10 Sekunden (initial_boot_delay_ms), bevor es in den Deep Sleep geht, damit du im Notfall immer ein neues Update einspielen kannst.

📐 Schritt 4: Sensoren kalibrieren (Wichtig!)
Da jedes Board und jede Erde leichte Fertigungstoleranzen aufweisen, musst du die Rohwerte für Bodenfeuchte und Düngergehalt (EC) einmalig ermitteln.

1. Kalibrierungsmodus aktivieren
Öffne die ESPHome-Weboberfläche des Boards oder dein Home Assistant Dashboard.

Aktiviere den Schalter "System: Kalibrierungsmodus (Rohwerte)".

Effekt: Der Deep Sleep wird blockiert und das System loggt alle 2 Sekunden die unberührten Volt-Rohwerte direkt in die Konsole.

2. Bodenfeuchte kalibrieren
Trockenwert (0%): Lass den Sensor komplett trocken an der Luft liegen. Öffne das ESPHome-Log und beobachte die Ausgabe. Note den stabilen Volt-Wert (z.B. 2.80 V).
Trage diesen im Hauptskript bei soil_moist_raw_dry ein.

Nasswert (100%): Tauche den Sensor bis zur oberen weißen Markierung in ein Glas Wasser. Notiere den Volt-Wert (z.B. 1.60 V). Trage diesen bei soil_moist_raw_wet ein.

3. Düngergehalt / EC-Wert kalibrieren
Der EC-Sensor nutzt eine präzise 5-Punkt-Leitlinie. Aktiviere den Kalibrierungsmodus und miss die Volt-Werte für folgende Zustände:

P0 (Minimalwert): Sensor komplett trocken oder in reinem Regenwasser -> Volt-Wert bei soil_ec_raw_p0 eintragen.

P1 (Sehr nährstoffarm): In feuchte, ungedüngte Torferde stecken -> Wert bei soil_ec_raw_p1 eintragen.

P2 (Leicht gedüngt): In feuchte Anzuchterde stecken -> Wert bei soil_ec_raw_p2 eintragen.

P3 (Optimaler Bereich): In feuchte, gut versorgte Qualitäts-Gartenerde stecken -> Wert bei soil_ec_raw_p3 eintragen.

P4 (Maximalwert): In eine gesättigte Kochsalzlösung tauchen -> Wert bei soil_ec_raw_p4 eintragen.

4. Werte übernehmen
Trage deine ermittelten Werte im Hauptskript t-higrow-main.yaml im Bereich substitutions: unter Kalibrierung ein und flashe das Board erneut. Deaktiviere danach den Kalibrierungsmodus, damit der Deep Sleep wieder greift.
