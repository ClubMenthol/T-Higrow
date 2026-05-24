==============================================================================
README: Cannabis Monitor (Garten/Indoor) - ESPHome Konfiguration
==============================================================================

PROJEKT-DETAILS:
  + Hardware:  LILYGO T-Higrow ESP32 (V1.1 mit DHT11)
  + Software:  ESPHome
  + Version:   4.3.0
  + Dateiname: t-higrow-final.yaml

BESCHREIBUNG:
  Dieses Skript dient der detaillierten Überwachung von Boden- und Klima-
  werten. Es ist speziell auf die Phasen der Pflanzenzucht (z.B. Cannabis)
  abgestimmt und berechnet komplexe Werte wie das Dampfdruckdefizit (VPD)
  und den Taupunkt. Dank Deep-Sleep-Logik ist es für den Akkubetrieb im
  Garten optimiert.

==============================================================================
ÄNDERUNGSHISTORIE (CHANGELOG):
==============================================================================
v4.3.0 (Aktuell):
  * NEU: Dedizierter Kalibrierungsmodus via virtuellem Schalter hinzugefügt.
  * NEU: Intervall-Schleife (2s) loggt unfiltrierte Rohwerte (Volt) direkt
         in die Web-Konsole, sobald der Kalibrierungsmodus aktiv ist.
  * Optimierung: Filterketten vereinfacht und für schnellere Messwerterfassung
    angepasst (Median-Filter über definierte Samples).

==============================================================================
STRUKTUR DER KONFIGURATION (MODULE):
==============================================================================
Modul 1: Variablen & Schwellenwerte
  -> Am Anfang des Skripts! Hier werden WLAN, IP-Adressen, Passwörter,
     Deep-Sleep-Zeiten sowie die Kalibrierungsdaten für Sensoren gepflegt.

Modul 2: System & Grundkonfiguration
  -> Steuert den Boot-Vorgang. Schaltet die Stromversorgung der Sensoren
     (Pin 4) vor dem I2C-Bus ein, um Blockaden zu verhindern. Regelt zudem
     die Bedingungen für den automatischen Deep Sleep im Loop.

Modul 3: Netzwerk & Schnittstellen
  -> Regelt WiFi (inkl. Fallback-AP "Garten-Fallback"), statische IP-Vergabe,
     mDNS, Webserver (Port 80), Native API (verschlüsselt) und MQTT-Broker.

Modul 4: Steuerung (Schalter & Buttons)
  -> Virtuelle Schalter für "Wach bleiben" (OTA-Updates) und "Kalibrierungsmodus".
     Physischer Schalter für die Sensor-Power und Hardware-Wake-Taster (Pin 35).

Modul 5: Physische & Berechnete Sensoren
  -> Boden: Feuchtigkeit (ADC 32) & Leitfähigkeit/EC-Wert (ADC 34) mit
     Kurven-Mapping.
  -> Klima: Temperatur & relative Feuchte (DHT11), Lichtintensität (BH1750 via I2C).
  -> Berechnungen: Absolute Luftfeuchte, Taupunkt und VPD (Dampfdruckdefizit).
  -> Energie: Akku-Spannungsteiler-Kompensation (ADC 33) mit Prozent-Umrechnung.

Modul 6: Text-Sensoren & Auswertungen
  -> Wandelt numerische Sensorwerte in klare Text-Zustände um (z. B. "Optimal",
     "Zu trocken", "Lichtstress", "VPD-Phasenempfehlungen" von Sämling bis Blüte).

Modul 7: Intervall-Routinen (Logging)
  -> Übernimmt das automatische Debug-Logging der reinen Volt-Rohwerte der
     Bodensensoren bei aktiver Kalibrierung.

==============================================================================
ANWENDUNGSHINWEISE & INBETRIEBNAHME:
==============================================================================

1. ANPASSUNG VOR DEM FLASHEN:
   Öffne die *.yaml Datei und passe in "MODUL 1" (Substitutions) zwingend deine
   Netzwerkdaten an:
   - wifi_ssid / wifi_password
   - manual_static_ip / manual_gateway / manual_dns1
   - mqtt_broker

2. ERSTE KALIBRIERUNG (BODENSENSOREN):
   Um genaue Prozentwerte für die Bodenfeuchte und mS/cm für den EC-Wert zu
   erhalten, gehe wie folgt vor:
   a) Flashe das Skript auf den ESP32.
   b) Rufe das Webinterface auf oder beobachte das Log im ESPHome-Dashboard.
   c) Aktiviere den Schalter "System: Kalibrierungsmodus (Rohwerte)".
   d) Lies die Volt-Werte im Log ab:
      - Sensor trocken in der Luft -> Wert für 'soil_moist_raw_dry'
      - Sensor in Wasser getaucht   -> Wert für 'soil_moist_raw_wet'
   e) Verfahre ebenso für die EC-Schwellenwerte mit entsprechenden Referenzen.
   f) Trage die ermittelten Volt-Werte oben in Modul 1 ein und flashe erneut.

3. DEEP-SLEEP BLOCKIEREN FÜR WARTUNG:
   Soll ein Update über WLAN (OTA) durchgeführt werden, muss im Webinterface
   oder via Home Assistant / MQTT der Schalter "System: Wach bleiben (Deep Sleep)"
   aktiviert werden. Andernfalls legt sich das Board nach Ablauf der Zeit
   ('initial_boot_delay_ms') sofort wieder schlafen.
==============================================================================
