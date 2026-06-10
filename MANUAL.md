# T-Higrow Cannabis Monitor

1. Die Kernkomponenten im Hauptskript
Das Hauptskript deklariert die globalen, persistenten Variablen (globals), die auch tiefe Schlafphasen (Deep Sleep) im Flash-Speicher überdauern:
* current_temp_offset (float): Flexibler Temperaturkorrekturwert.
* is_powered (bool): Statusflag für externe Stromversorgung.
* accum_heat_stress_min (float): Speicher für akkumulierte Stressminuten.
* accum_dli (float): Laufender Tagesspeicher für die Lichtsumme.

2. Signalverarbeitung & Filter-Pipeline
Analoge Sensoren an Mikrocontrollern leiden im Realbetrieb unter hochfrequentem Rauschen, Spannungsschwankungen und transienten Spikes (Fehlmessungen). Um eine Verfälschung von Langzeitstatistiken in Home Assistant zu verhindern, durchlaufen alle kritischen Analogwerte eine zweistufige Filterkette.
Plaintext[ 
Physischer ADC-Pin ] 
         │
         ▼
 1. MEDIAN-FILTER  ──► Eliminiert extreme Ausreißer & Spikes
         │
         ▼
 2. GLEITENDER MITTELWERT ──► Glättet das Signal & drosselt Sendeintervall
         │
         ▼
[ Home Assistant / MQTT ]

Phase 1: Der Median-Filter (median)
Der Median-Filter sammelt ein definiertes Fenster an Rohwerten (z. B. median_window: "5"). Er sortiert diese Werte nach ihrer Größe und gibt exakt den mittleren Wert weiter. Extreme Ausreißer nach oben oder unten (z. B. durch kapazitive Einstreuungen oder Schaltnetzteile) werden mathematisch vollständig eliminiert.
Phase 2: Der gleitende Durchschnitt (sliding_window_moving_average)
Nachdem die Kurve von Spikes befreit wurde, berechnet dieser Filter den arithmetischen Mittelwert über ein größeres Fenster (z. B. ma_window: "30"). Er sorgt für einen absolut glatten, harmonischen Kurvenverlauf. Durch die Konfiguration von send_every: "30" wird das Signal erst dann an Home Assistant übertragen, wenn das Fenster voll ist. Dies reduziert die Netzwerklast und verhindert das Aufblähen der Home Assistant Datenbank.

3. Agrarwissenschaftliche & Klimatische Berechnungen
Ein zentrales Feature des Monitors ist die autarke Berechnung komplexer pflanzenphysiologischer Indikatoren direkt auf dem ESP32.

Dampfdruckdefizit (VPD)
Das Vapor Pressure Deficit (gemessen in kPa) beschreibt den Unterschied zwischen dem Sättigungsdampfdruck an der Blattoberfläche und dem tatsächlichen Dampfdruck der Umgebungsluft. Das System nutzt die Tetens-Formel zur Berechnung des Sättigungsdampfdrucks ($SVP$):
$$SVP = 0.61078 \times \exp\left(\frac{17.27 \times T}{T + 237.3}\right)$$

Daraus wird über die relative Luftfeuchtigkeit ($H$) der tatsächliche Dampfdruck ($AVP$) ermittelt:
$$AVP = \frac{SVP \times H}{100.0}$$
Das Defizit ergibt sich aus: $VPD = SVP - AVP$. Ein Template-Textsensor gleicht das Ergebnis permanent mit deinen biologischen Schwellenwerten ab und gibt den Status im Klartext aus (z. B. "Ideal für: Wachstumsphase" oder "Achtung: Trockenstress!").
Daily Light Integral (DLI)Das DLI gibt an, wie viele Mole der Photonen aus dem photosynthetisch aktiven Lichtspektrum (PAR) pro Quadratmeter über den gesamten Tag verteilt auf die Pflanze treffen ($mol/m²/d$).
1. PPFD-Schätzung: Da der BH1750 Lux liefert, wird der Wert über einen empirischen Faktor für Sonnenlicht/Breitband-LEDs umgerechnet:
$$PPFD = Lux \times 0.0185 \quad (\mu mol/m²/s)$$
2. Minütliche Akkumulation: Das System fragt den Lichtwert alle 60 Sekunden ab, berechnet die Stoffmenge pro Minute und addiert sie auf:
$$mol/m²/Minute = \frac{PPFD \times 60.0}{1.000.000}$$
Dieser Wert wird zyklisch in accum_dli gesichert. Um Punkt 00:00 Uhr setzt eine SNTP-zeitgesteuerte Routine den Wert automatisch wieder auf 0.0.
Schimmel-Risiko & TaupunktÜber die Magnus-Formel berechnet das Modul erweitert.yaml fortlaufend den exakten Taupunkt. Nähert sich die Umgebungstemperatur dem Taupunkt auf weniger als 3°C an und liegt die relative Luftfeuchtigkeit gleichzeitig über 75%, schaltet der Sensor text_schimmel_risiko sofort auf "AKUTE GEFAHR (Kondensat)".

4. Energiemanagement & Thermische Kompensation
Das System beherrscht zwei grundlegend verschiedene Betriebsmodi, die vollautomatisch anhand der anliegenden Akkuspannung (Pin 33) unterschieden werden:
A. Reiner Akkubetrieb (Deep Sleep)Liegt die gemessene Spannung unter 4.3V, läuft das System im extrem stromsparenden Intervallbetrieb. Das Board startet, aktiviert über das Sensor-Power-Gate (GPIO 4) die Peripherie, wartet auf die Stabilisierung der Filterketten, sendet die Daten via MQTT und geht sofort für 15 Minuten in den Tiefschlaf (Deep Sleep). Da das Board hierbei nur wenige Sekunden aktiv ist, findet kaum eine Eigenerwärmung statt. Der Temperatur-Offset ist minimal eingestellt:YAMLtemp_offset_battery: "-0.1"
B. Dauerstrom- / Ladebetrieb (Dauerbetrieb)Sobald ein USB-C-Kabel oder Ladegerät angeschlossen wird, steigt die Spannung am ADC über 4.3V. Das System erkennt dies im Modul energie.yaml und setzt das Flag id(is_powered) = true.
Der Deep Sleep wird ausgesetzt, das Board bleibt permanent online (erreichbar für OTA-Updates und Live-Messungen).
Durch den dauerhaften Betrieb des ESP32-Chips und der WLAN-Endstufe erwärmt sich die Platine massiv. Diese Abwärme strahlt direkt auf den On-Board DHT11-Sensor ab.
Kompensation: Das System wechselt die Korrekturvariable augenblicklich auf den dauerstrom-optimierten Wert:
temp_offset_powered: "-4.5"
Dadurch bleiben die an Home Assistant übermittelten Raumklimadaten auch im USB-Betrieb präzise.

5. Multi-Board-Verwaltung (MAC-Mapping)
Um den Pflegeaufwand bei der Nutzung mehrerer Monitore gegen Null zu senken, implementiert das Modul pflanzen_mapping.yaml eine automatisierte Identifikation über die Hardware-Adresse des Mikrocontrollers.
Im Substitution-Block oder in den Secrets werden die Kurz-MACs (die letzten 6 Stellen der MAC-Adresse im Kleinformat) sowie der Wunschname registriert:

mac_p1: "a806a0"
name_p1: "Frisian Dew"

mac_p2: "a806a2"
name_p2: "Karls Beste"

Beim Systemstart führt das Board ein C++ Lambda aus:
std::string current_device = esphome::App.get_name();
if (current_device.find("${mac_p1}") != std::string::npos) {
  return {"${name_p1}"};
}
Das Board erkennt sich selbst, benennt die Home Assistant Entitäten dynamisch nach der zugewiesenen Pflanzensorte und verhindert jegliche Namenskonflikte im Netzwerk.

6. Ausführliche Kalibrierungsanleitung

Die im Gehirn des ESP32 verbauten Analog-Digital-Wandler (ADC) weisen fertigungsbedingte Toleranzen auf. Für verlässliche Werte musst du die Sensoren einmalig kalibrieren.
Schritt 1: Aktivierung des Wartungsmodus
Schalte im Home Assistant Dashboard oder auf der ESPHome-Weboberfläche deines Boards den Schalter "System: Kalibrierungsmodus (Rohwerte)" ein.
Dieser Schalter deaktiviert temporär die mathematischen Kurven-Berechnungen und die Filter-Pipeline.
Ein dedizierter Intervall-Logger feuert nun alle 2 Sekunden die unberührten Volt-Rohwerte direkt in die Konsole:

[INFO] --- KALIBRIERUNG AKTIV --- Feuchte: 2.145 V | EC-Wert: 0.284 V

Schritt 2: Kalibrierung der Bodenfeuchte (Kapazitiv)
Der Sensor misst die Dielektrizitätskonstante des Bodens. Die Werte verhalten sich umgekehrt proportional (höhere Spannung = trockener).
1. Trockenpunkt (0%): Säubere den Sensor vollständig und lass ihn trocken an der Luft liegen. Lies den Volt-Wert in der Konsole ab (z. B. 2.471 V). Trage diesen Wert in deine secrets.yaml unter soil_moist_raw_dry ein.
2. Nasspunkt (100%): Tauche den Sensor bis zur maximalen weißen Eintauchlinie in ein Glas Wasser (Vermeide Kontakt mit dem Glasrand). Notiere den Volt-Wert (z. B. 1.372 V) und trage ihn unter soil_moist_raw_wet in die Secrets ein.

Schritt 3: Kalibrierung der elektrischen Leitfähigkeit (EC / Dünger)
Da sich die Leitfähigkeit in Erde nicht linear verhält, nutzt dieses Projekt eine hochpräzise 5-Punkt-Kennlinie (p0 bis p4). Das Skript interpoliert die Werte im Realbetrieb dynamisch zwischen diesen Stützpunkten.
Ermittle nacheinander die Spannungen für folgende Zustände und pflege die Rohwerte in deine secrets.yaml ein:
* p0 (0.0 mS/cm — Minimalwert): Sensor vollkommen trocken in der Luft oder eingetaucht in reinem, destillierten Wasser / sauberem Regenwasser.
* p1 (0.3 mS/cm — Sehr nährstoffarm): Sensor in vollständig durchnässter Torferde oder ungedüngter Kokosfaser.
* p2 (0.8 mS/cm — Leicht gedüngt): Sensor in nasser, leicht vorgedüngter Anzuchterde.
* p3 (1.5 mS/cm — Optimalbereich): Sensor in gut gesättigter, vollaktiver Universalerde / vorgedüngter Grow-Erde (z. B. Plagron/BioBizz im optimalen Nährstofffenster).
* p4 (5.0 mS/cm — Maximalwert): Sensor eingetaucht in eine gesättigte Kochsalzlösung (Wasser, in dem sich kein Salz mehr auflösen lässt). Das markiert den absoluten Maximalausschlag des Sensors.
Nachdem alle Werte in der secrets.yaml hinterlegt sind, flashst du das Board neu und deaktivierst den Kalibrierungsmodus.

7. Integration in Home Assistant & Wartung
Over-The-Air (OTA) Updates im Intervallbetrieb
Da das Board im Akkubetrieb fast dauerhaft schläft, ist es für kabellose Updates regulär nicht erreichbar. Um ein Update einzuspielen, gehst du wie folgt vor:
1. Warte, bis das Board kurz aufwacht, oder drücke den physischen Wake-Taster (Pin 35) auf der Platine, um den ESP32 manuell aufzuwecken.
2. Schalte im Home Assistant Dashboard sofort den Schalter "System: Wach bleiben (Deep Sleep)" ein.
3. Das Board blockiert nun den Tiefschlaf dauerhaft beim nächsten Loop-Durchlauf.
4. Führe dein OTA-Update über das ESPHome Dashboard komfortabel durch.
5. Schalte den "Wach bleiben"-Switch wieder aus, um das Board zurück in den batterieschonenden Zyklus zu schicken.
