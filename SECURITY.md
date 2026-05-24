# Sicherheitsrichtlinien (Security Policy)

## Unterstützte Versionen

Derzeit wird nur die jeweils aktuellste Version im `main`-Branch aktiv mit Sicherheitsupdates und Bugfixes versorgt.

| Version | Unterstützt |
| ------- | ----------- |
| v4.3.x  | ✅ Ja       |
| < v4.3  | ❌ Nein      |

## Melden einer Sicherheitslücke

**WICHTIG: Wenn du versehentlich deine `secrets.yaml` oder echte Passwörter in dieses Repository oder in ein öffentliches Issue hochgeladen hast:**

1. Ändere **sofort** dein WLAN-Passwort und die API-Schlüssel deines lokalen ESP32-Gerichts!
2. Lösche den Commit nicht einfach nur lokal, da er in der Git-Historie auf GitHub sichtbar bleibt.
3. Nutze Tools wie `git-filter-repo` oder den GitHub-Support, um sensible Daten vollständig aus der Historie zu entfernen.

Wenn du eine generelle Schwachstelle im Code (z. B. in den Berechnungs-Lambdas) findest, erstelle bitte ein privates Issue oder kontaktiere den Repository-Inhaber direkt, an
