# Swatted-Server mit AMP

Diese Dateien sind ein Template für [AMP von CubeCoders](https://cubecoders.com/AMP) (Modul „Generic“). Damit läuft
die Multiplayer-Lobby von Swatted auf einem eigenen Server: ohne Host-Spieler, ohne Portfreigabe bei den Spielern.
Der Server ist dasselbe Spiel wie der Client, nur ohne Fenster (`--headless -- --server`).

| Datei | Inhalt |
|---|---|
| `swatted.kvp` | Programm, Startzeile, Stopp-Befehl `quit`, Erkennung von „Server bereit“, Beitreten und Verlassen |
| `swattedconfig.json` | Einstellungen in AMP: Server, Einsatz, Download |
| `swattedports.json` | UDP-Port 47250 |
| `swattedupdates.json` | „Update“: Server-Archiv von deiner Download-Adresse holen und entpacken |
| `swattedmetaconfig.json` | leer (der Server hat keine Konfigurationsdatei) |
| `manifest.json` | Kennung des Template-Repositorys `Lappen32/Swatted-Server` (siehe Schritt 2) |

**Stand:** Server-Start, Konsole, Log und Spiel gegen Bots sind lokal getestet (`tests/dedicated_server_test`, auch
mit dem exportierten Windows-Build). Mit einer echten AMP-Instanz ist das Template noch nicht ausprobiert.

## 1. Server-Archiv besorgen

Jeder GitHub-Release enthält zwei Archive, in denen das Programm direkt oben liegt:

* `Swatted-<Version>-server-linux.zip` – `Swatted.x86_64` (für AMP unter Linux, auch im Docker-Container)
* `Swatted-<Version>-server-windows.zip` – `Swatted.exe` und `Swatted.console.exe` (für AMP unter Windows)

Das Repository ist privat, AMP kann diese Archive deshalb nicht direkt von GitHub laden. Zwei Wege:

* **Download-Adresse:** Das Archiv irgendwo ablegen, wo es ohne Anmeldung herunterzuladen ist, z. B. als
  Nextcloud-Freigabe („Link teilen“, an den Link `/download` anhängen). Diese Adresse trägst du in AMP ein.
* **Manuell:** Das Archiv mit dem Dateimanager von AMP hochladen (Schritt 4).

## 2. Template in AMP einbinden

AMP lädt Templates aus GitHub-Repositorys. Das Swatted-Template liegt in
[Lappen32/Swatted-Server](https://github.com/Lappen32/Swatted-Server), nur diese Dateien, nichts vom Spiel. AMP hat
dort kein Feld für Zugangsdaten, das Repository muss also **öffentlich** sein.

1. In AMP (ADS-Instanz): **Configuration → Instance Deployment → Configuration Repositories**, dort
   `Lappen32/Swatted-Server:main` eintragen.
2. **Fetch** klicken, dann die Seite im Browser neu laden.

**Template ändern:** Gepflegt wird es im Spiel-Repository unter `tools/amp/`; dort prüft `tests/dedicated_server_test`
es gegen den Server. Nach einer Änderung alle Dateien dieses Ordners außer `.gdignore` ins Hauptverzeichnis von
Swatted-Server kopieren (ohne Unterordner), dort committen, dann in AMP noch einmal **Fetch**.

## 3. Instanz anlegen

**Create Instance** → als Anwendung **Swatted** wählen → anlegen und öffnen.

## 4. Einstellungen

Unter **Configuration → Swatted**:

* **Server:**
  * Servername
  * Passwort (leer = keins)
  * Plätze (2–4)
  * optional die öffentliche IPv4: dann nennt das Log zusätzlich eine 6-stellige Lobby-ID
  * **Einstellungen sperren:** an = Karte und Einsatz legt nur der Server fest; aus = der Lobby-Leiter wählt sie im Spiel
  * UPnP: nur für einen Server zu Hause hinter einem Router
  * „Zurück zur Lobby nach“: so viele Sekunden nach einem Einsatz geht es von selbst in die Lobby, falls der Leiter
    nichts wählt
* **Einsatz:** die Startwerte der Lobby: Karte (`practice` oder `random`), Einsatz, Gegner, Stärke, Tageszeit,
  Jahreszeit, Friendly Fire.
* **Download:**
  * **„Von der Download-Adresse“** + die Adresse aus Schritt 1 (Linux bzw. Windows), dann **Update**.
  * **„Manuell hochladen“**: Im **File Manager** in den Ordner `swatted` wechseln, das Server-Archiv hochladen,
    entpacken (Rechtsklick → Extract) und das Archiv löschen. Danach einmal **Update**: Das macht unter Linux das
    Programm ausführbar.

## 5. Starten

**Start.** In der Konsole erscheint `Server bereit: …`, AMP zeigt den Server dann als laufend. Spieler, die beitreten
oder gehen, zählt AMP mit.

Der Port ist **UDP 47250**. AMP zeigt ihn unter **Network** und kann ihn ändern. In der Firewall des Servers (und bei
einem Server zu Hause am Router) muss dieser UDP-Port offen sein.

## 6. Mitspielen

Im Spiel: **Multiplayer → Beitreten**, als Lobby-Code `Adresse:Port` des Servers eingeben (z. B.
`203.0.113.7:47250` oder `swat.example.de:47250`), dazu das Passwort. Oben in der Lobby steht der Servername.

* **Lobby-Leiter:** Wer am längsten in der Lobby ist, ist Lobby-Leiter (in der Liste „Leiter“). Er wählt Karte und
  Einsatz, kann Spieler entfernen und entscheidet nach dem Einsatz über Neustarten, Nächste Map oder Lobby.
* **Start:** Sind mehr als die Hälfte bereit, startet der Einsatz nach dem Countdown.
* **Version:** Server und Spieler brauchen **dieselbe Spielversion**, sonst lehnt der Server mit „Andere Spielversion“
  ab.

## 7. Konsole

| Befehl | Wirkung |
|---|---|
| `help` | alle Befehle |
| `status` | Zustand, Bildrate, Einstellungen, Spieler |
| `spieler` | Spieler mit Slot, Leiter, bereit |
| `karten` | spielbare Karten (IDs für `set karte`) |
| `set <wert> <wahl>` | z. B. `set gegner viele`, `set karte random`, `set modus geiseln`, `set ff aus`, `set plätze 3` |
| `start` | Einsatz sofort starten |
| `neustart` / `weiter` | Einsatz neu starten / nächste Karte |
| `lobby` | laufenden Einsatz beenden, alle zurück in die Lobby |
| `kick <slot\|name>` | Spieler entfernen |
| `passwort <text\|->` | Passwort setzen, `-` entfernt es |
| `sperren an\|aus` | nur der Server ändert die Einstellungen |
| `quit` | Server beenden; die Spieler sehen „Der Server wurde beendet.“ (das macht auch AMPs Stop) |

## 8. Neue Spielversion

1. Neues Server-Archiv an dieselbe Download-Adresse legen bzw. hochladen.
2. **Update**.
3. Server neu starten.

## Probleme

* **„Keine Verbindung zum Host“** beim Beitreten: Der UDP-Port ist nicht offen (Firewall, Router), oder Adresse bzw.
  Port stimmen nicht. Den Port zeigt AMP unter Network, die Konsole in der Zeile `Server bereit`.
* **AMP bleibt bei „Starting“:** In der Konsole nachsehen. Steht dort „kein freier UDP-Port“, ist der Port belegt.
  Fehlt das Programm, war das Update ohne Archiv (Download-Adresse prüfen oder manuell hochladen).
* **Linux: „Permission denied“:** Einmal **Update**; das setzt das Ausführungsrecht.
