# Beitragen zum ESPHome Cannabis Monitor

Vielen Dank, dass du dieses Projekt verbessern möchtest! Um die Zusammenarbeit so sauber und effizient wie möglich zu gestalten, beachte bitte die folgenden Richtlinien.

## Wie kann ich beitragen?

### Fehler melden (Issues)
* Nutze die vorhandene Issue-Vorlage für Bug Reports.
* Stelle sicher, dass der Fehler nicht bereits in einem anderen Issue diskutiert wird.
* Hänge immer ein anonymisiertes ESPHome-Log an, wenn es sich um ein Laufzeitproblem handelt.

### Code-Änderungen einreichen (Pull Requests)
1. Erstelle einen Fork dieses Repositories.
2. Erstelle einen neuen Branch für deine Änderungen (`git checkout -b feature/tolles-neues-feature`).
3. Achte darauf, dass deine YAML-Struktur valide ist (`esphome config t-higrow-final.yaml`).
4. Kommentiere Änderungen im Code verständlich auf Deutsch.
5. Erhöhe bei funktionalen Änderungen die Versionsnummer im Datei-Header (z. B. von v4.3.1 auf v4.3.2).
6. Sende einen Pull Request gegen den `main`-Branch dieses Repositories.

## Code-Stil & Richtlinien
* **Modularität:** Halte die Struktur strikt in den vorgegebenen Modulen (Modul 1 bis Modul 7) getrennt.
* **Geheimnisschutz:** Füge niemals echte Zugangsdaten in die Hauptdatei ein. Nutze ausnahmslos das `!secret`-System.
