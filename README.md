# Swatted server with AMP

These files are a template for [AMP by CubeCoders](https://cubecoders.com/AMP) (the "Generic" module). With it, the
multiplayer lobby of Swatted runs on a server of your own: nobody has to host, and no player has to open a port. The
server is the same game as the client, only without a window (`--headless -- --server`).

The game is in German, and so are the server's log, its console answers and the template's settings in AMP. This
guide quotes them as they appear, with their meaning next to them.

| File | Contents |
|---|---|
| `swatted.kvp` | program, command line, stop command `quit`, detection of "Server bereit" (ready), joins and leaves |
| `swattedconfig.json` | the settings in AMP: Server, Einsatz (mission), Download |
| `swattedports.json` | UDP port 47250 |
| `swattedupdates.json` | "Update": fetch the server archive from your download address and unpack it |
| `swattedmetaconfig.json` | empty (the server has no config file) |
| `manifest.json` | identifies this template repository `Lappen32/Swatted-Server` (see step 2) |

**Status:** server start, console, log and a game against bots are tested locally (`tests/dedicated_server_test` in
the game's repository, also with the exported Windows build). The template has not yet been tried on a real AMP
instance.

## 1. Get the server archive

Every release of the game contains two archives with the program at the top level:

* `Swatted-<version>-server-linux.zip` – `Swatted.x86_64` (for AMP on Linux, also inside its Docker container)
* `Swatted-<version>-server-windows.zip` – `Swatted.exe` and `Swatted.console.exe` (for AMP on Windows)

The game's repository is private, so AMP cannot download these archives from GitHub. Two ways:

* **Download address:** put the archive where it can be downloaded without a login, e.g. a Nextcloud share ("Share
  link", then append `/download` to the link). Enter that address in AMP.
* **Manual:** upload the archive with AMP's file manager (step 4).

## 2. Add the template to AMP

AMP loads templates from GitHub repositories, such as this one. It holds only the template files, nothing of the
game. AMP has no field for credentials there, so this repository must stay **public**.

1. In AMP (the ADS instance): **Configuration → Instance Deployment → Configuration Repositories**, add
   `Lappen32/Swatted-Server:main`.
2. Click **Fetch**, then reload the page in your browser.

**Changing the template:** the original lives in the game's repository under `tools/amp/`, where
`tests/dedicated_server_test` checks it against the server. After a change, copy every file of that folder except
`.gdignore` to the top level of this repository (no subfolder), commit, and click **Fetch** in AMP again.

## 3. Create the instance

**Create Instance** → choose **Swatted** as the application → create and open it.

## 4. Settings

Under **Configuration → Swatted**:

**Server**

| Setting | Meaning |
|---|---|
| Servername | the server's name, shown at the top of the lobby |
| Passwort | the password to join; empty = none |
| Plätze | player slots (2–4) |
| Öffentliche Adresse | optional, the public IPv4: the log then also shows a 6-character lobby code |
| Einstellungen sperren | on = only the server sets map and mission; off = the lobby leader picks them in the game |
| UPnP | only for a server at home behind a router: opens the port there |
| Zurück zur Lobby nach | seconds after a mission until everyone is back in the lobby, unless the leader chooses first (0 = never) |

**Einsatz** (mission): the lobby's starting values – Karte (map: `practice` or `random`), Einsatz (mode: Jagd = hunt,
Bombe = bomb, Geiseln = hostages), Gegner (enemies), Stärke (skill), Tageszeit (time of day), Jahreszeit (season),
Friendly Fire.

**Download**

* **Von der Download-Adresse** (from the download address): enter the address from step 1 (Linux or Windows), then
  click **Update**.
* **Manuell hochladen** (upload by hand): in the **File Manager**, open the folder `swatted`, upload the server
  archive, unpack it (right-click → Extract) and delete the archive. Then click **Update** once: on Linux this makes
  the program executable.

## 5. Start

Click **Start**. The console shows `Server bereit: …` (server ready), and AMP then shows the server as running. AMP
counts the players who join or leave.

The port is **UDP 47250**. AMP shows it under **Network** and can change it. This UDP port must be open in the
server's firewall (and, for a server at home, in the router).

## 6. Playing

In the game: **Multiplayer → Beitreten** (join), enter the server's `address:port` as the lobby code (e.g.
`203.0.113.7:47250` or `swat.example.com:47250`) and the password. The server's name is shown at the top of the lobby.

* **Lobby leader:** whoever has been in the lobby the longest is the lobby leader (marked "Leiter" in the list). The
  leader picks map and mission, can remove players, and after a mission chooses between restart, next map and lobby.
* **Start:** once more than half of the players are ready, the mission starts after the countdown.
* **Version:** the server and the players need **the same game version**; otherwise the server refuses with "Andere
  Spielversion" (different game version).

## 7. Console

The commands work in English and in German; the answers are German.

| Command | Effect |
|---|---|
| `help` | all commands |
| `status` | state, frame rate, settings, players |
| `players` | players with slot, leader, ready |
| `maps` | playable maps (the IDs for `set map`) |
| `set <option> <value>` | e.g. `set enemies many`, `set map random`, `set mode hostages`, `set ff off`, `set max-players 3` |
| `start` | start the mission now |
| `restart` / `next` | restart the mission / next map |
| `lobby` | end the running mission, everyone back to the lobby |
| `kick <slot\|name>` | remove a player |
| `password <text\|->` | set the password; `-` removes it |
| `lock on\|off` | on = only the server changes the settings |
| `quit` | stop the server; the players see "Der Server wurde beendet." (the server was shut down). AMP's Stop does the same. |

Options for `set`: `map`, `mode` (hunt / bomb / hostages), `enemies` (none / few / normal / many), `skill` (easy /
normal / hard), `time` (day / night / random), `season` (map / spring / summer / autumn / winter), `ff` (on / off),
`max-players` (2–4), and for random maps `size`, `indoor`, `traps` (a step from 0, as on the game's sliders).

## 8. New game version

1. Put the new server archive at the same download address, or upload it.
2. Click **Update**.
3. Restart the server.

## Troubleshooting

* **"Keine Verbindung zum Host"** (no connection to the host) when joining: the UDP port is not open (firewall,
  router), or the address or port is wrong. AMP shows the port under Network, the console in the line `Server bereit`.
* **AMP stays at "Starting":** look at the console. "kein freier UDP-Port" (no free UDP port) means the port is taken.
  If the program is missing, Update ran without an archive (check the download address, or upload it by hand).
* **Linux: "Permission denied":** click **Update** once; it sets the execute permission.
