## 📝 Beschreibung
Eine kurze Beschreibung der Änderungen oder des neu hinzugefügten Features. 

*(z. B. Umstellung der Template-Sensoren auf C++ Lambdas zur Behebung von ID-Parse-Fehlern)*

## 🏷️ Versions-Update
- **Alte Version:** vX.X.X
- **Neue Version:** vX.X.X

## 🛠️ Betroffene Module / Komponenten
Bitte ankreuzen, welche Bereiche der Konfiguration geändert wurden:
- [ ] MODUL 1: Variablen & Schwellenwerte (Substitutions)
- [ ] MODUL 2: System & Grundkonfiguration (Deep Sleep, Boot)
- [ ] MODUL 3: Netzwerk & Schnittstellen (WLAN, MQTT, API)
- [ ] MODUL 4: Steuerung (Schalter & Buttons)
- [ ] MODUL 5: Physische & berechnete Sensoren
- [ ] MODUL 6: Text-Sensoren & Auswertungen
- [ ] MODUL 7: Intervall-Routinen (Logging)

## ⚙️ Funktionaler Check
- [ ] `esphome config t-higrow-final.yaml` läuft fehlerfrei durch ("Configuration is valid!").
- [ ] Der Kompiliervorgang bricht nicht ab.
- [ ] Das Verhalten im Akkubetrieb (20s-Fenster) wurde berücksichtigt.
- [ ] Das Verhalten im Dauerbetrieb (Ladegerät angeschlossen) wurde berücksichtigt.

## 📊 Test-Log / Validierung
Bitte hier einen kurzen Ausschnitt aus dem MQTT- oder ESPHome-Log einfügen, der belegt, dass die Sensoren plausibel und im gewünschten Intervall senden:

```text
// Hier Log-Ausschnitt einfügen
