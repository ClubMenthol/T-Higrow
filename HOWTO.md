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

```

## 🚀 Schritt 3: Firmware flashen
Verbinde das T-Higrow-Board per USB-C mit deinem Computer. Nutze das ESPHome-Dashboard oder das Terminal, um das Skript zu kompilieren und zu installieren.

Befehl für das Terminal:
**esphome run t-higrow-main.yaml**

Hinweis: Beim ersten Booten wartet das Board standardmäßig 10 Sekunden (initial_boot_delay_ms), bevor es in den Deep Sleep geht, damit du im Notfall immer ein neues Update einspielen kannst.

## 📐 Schritt 4: Sensoren kalibrieren (Wichtig!)
Da jedes Board und jede Erde leichte Fertigungstoleranzen aufweisen, musst du die Rohwerte für Bodenfeuchte und Düngergehalt (EC) einmalig ermitteln.

***1. Kalibrierungsmodus aktivieren***
Öffne die ESPHome-Weboberfläche des Boards oder dein Home Assistant Dashboard.

Aktiviere den Schalter "System: Kalibrierungsmodus (Rohwerte)".

Effekt: Der Deep Sleep wird blockiert und das System loggt alle 2 Sekunden die unberührten Volt-Rohwerte direkt in die Konsole.

***2. Bodenfeuchte kalibrieren***

### Den Trocken-Wert ermitteln (0 % Feuchtigkeit)
Dieser Wert definiert den Punkt, an dem die Erde für die Pflanze "tot" bzw. komplett ausgetrocknet ist.

Lass die Erde im Topf nun ganz normal austrocknen (oder nutze für eine schnellere Kalibrierung eine separate Schale mit absolut staubtrockener Erde aus demselben Sack).
oder
Warte bei einer bepflanzten Erde so lange, bis die Erde sichtbar trocken ist und deine Cannabis-Pflanzen gerade eben beginnen, die Blätter ganz leicht hängen zu lassen (der sogenannte permanente Welkepunkt).

Lies den Volt-Wert im Log ab. Da trockene Erde das elektrische Feld kaum leitet, ist die gemessene Spannung jetzt deutlich höher (typischerweise zwischen 2,50 V und 2,90 V).

Notiere dir diesen Wert für die Variable **soil_moist_raw_dry**.

### Den Nass-Wert ermitteln (100 % Feuchtigkeit)
Dieser Wert definiert die maximale Wasserkapazität deiner Erde, bevor sie anfängt zu "verschlammen".

* Stecke das Sensorboard bis zur vorgesehenen Markierung in deinen Pflanztopf oder das Beet.

* Gieße die Erde um den Sensor herum langsam und sehr kräftig. Gieße so lange, bis die Erde absolut gesättigt ist und das Wasser unten aus dem Topf abläuft (maximale Feldkapazität).

* Warte etwa 5 bis 10 Minuten, damit sich das Wasser gleichmäßig verteilt und überschüssiges Wasser versickert ist.

* Schaue in das Log deines ESPHome-Geräts. Der dort angezeigte Volt-Wert bei Feuchte (typischerweise ein niedriger Wert zwischen 1,35 V und 1,55 V) ist dein neuer Sättigungspunkt.

Notiere dir diesen Wert für die Variable **soil_moist_raw_wet**.

***3. Düngergehalt / EC-Wert kalibrieren***

Der EC-Sensor nutzt eine präzise 5-Punkt-Leitlinie. Aktiviere den Kalibrierungsmodus und miss die Volt-Werte für folgende Zustände:

P0 (Minimalwert): Sensor komplett trocken oder in reinem Regenwasser -> Volt-Wert bei soil_ec_raw_p0 eintragen.

P1 (Sehr nährstoffarm): In feuchte, ungedüngte Torferde stecken -> Wert bei soil_ec_raw_p1 eintragen.

P2 (Leicht gedüngt): In feuchte Anzuchterde stecken -> Wert bei soil_ec_raw_p2 eintragen.

P3 (Optimaler Bereich): In feuchte, gut versorgte Qualitäts-Gartenerde stecken -> Wert bei soil_ec_raw_p3 eintragen.

P4 (Maximalwert): In eine gesättigte Kochsalzlösung tauchen -> Wert bei soil_ec_raw_p4 eintragen.

***4. Werte übernehmen***
Trage deine ermittelten Werte im Hauptskript t-higrow-main.yaml im Bereich substitutions: unter Kalibrierung ein und flashe das Board erneut. Deaktiviere danach den Kalibrierungsmodus, damit der Deep Sleep wieder greift.
