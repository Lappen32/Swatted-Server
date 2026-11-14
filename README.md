# Swatted server with AMP

These files are a template for [AMP by CubeCoders](https://cubecoders.com/AMP) (the "Generic" module). With it, the
multiplayer lobby of Swatted runs on a server of your own: nobody has to host, and no player has to open a port. The
server is the same game as the client, only without a window (`--headless -- --server`).

The game is in German, and so are the server's log, its console answers and the template's settings in AMP. This
guide quotes them as they appear, with their meaning next to them.

| File | Contents |
|---|---|
| `swatted.kvp` | program, command line, stop command `quit`, detection of "Server bereit" (ready), joins and leaves, sleep mode |
| `swattedconfig.json` | the settings in AMP: Server (with the RAM limit), Einsatz (mission), Download |
| `swattedports.json` | UDP port 47250 |
| `swattedupdates.json` | "Update": fetch the server archive from your download address and unpack it |
| `swattedmetaconfig.json` | empty (the server has no config file) |
| `manifest.json` | identifies this template repository `Lappen32/Swatted-Server` (see step 2) |
| `CHANGELOG.md` | the template's versions (`Meta.ConfigVersion`) |

**Status:** server start, console, log, the RAM limit and a game against bots are tested locally
(`tests/dedicated_server_test` in the game's repository, also with the exported Windows build). On AMP 2.8 (Linux,
Docker) start, console and stop work; sleep mode (on again in template version 4) is not tried there yet.

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

**Changing the template:** this repository is the template's only copy (the game's repository no longer holds one).
The game's `tests/dedicated_server_test` checks a checkout of it against the server:
`godot --headless --path . res://tests/dedicated_server_test.tscn -- --amp=<path to this checkout>`. A change raises
`Meta.ConfigVersion` in `swatted.kvp` and gets a section in `CHANGELOG.md`; after the merge, update AMP (step 11).

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
| Maximaler RAM | the most memory the server should use, in MB; 0 = no limit (see step 7) |

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

## 6. Sleep mode

The template supports AMP's sleep mode, and it is **on** by default: once AMP has counted no player for 5 minutes, it
stops the server and listens on its UDP port itself. While the server sleeps it uses no CPU and no RAM.

* **AMP version:** use AMP 2.8.0.10 or newer, which fixed sleep bugs of the earlier 2.8 builds. CubeCoders have switched
  sleep mode off in their own templates for now (October 2026) while they test the reworked sleep; this template comes
  from a repository of its own and keeps it. Should it misbehave, switch it off in the instance (see "Settings" below).
* **Waking up:** the first player who joins wakes it. AMP starts the server, which takes a few seconds; the game keeps
  trying for up to 20 seconds and shows "Verbinde … Ein Server im Ruhezustand braucht ein paar Sekunden." (connecting -
  a sleeping server needs a few seconds), then the player is in the lobby. A game version without this retry gives up
  after 8 seconds: join again once the console shows `Server bereit`.
* **Back to sleep:** woken, but nobody joined within 2 minutes (the doze delay), the server sleeps again. Should a stray
  packet on the port (a port scan) wake it, the doze delay puts it back to sleep.
* **Counting players:** AMP counts them by the log lines `Beigetreten: <name> (n/max), ID <id>` (joined) and
  `Verlassen: <name> (n/max), ID <id>` (left). The ID keeps two players of the same name apart. A server build from
  before the ID logs the lines without it; the template still reads them, by name.
* **Settings:** switching sleep off, the delays and "sleep on start" are AMP's own settings of the instance (its sleep
  mode section). With sleep off the server keeps running; an empty lobby costs little CPU but its RAM (about 400 MB).

## 7. RAM limit

**Maximaler RAM** (MB) is passed as `--max-ram`. A Godot game cannot cap its own memory, so the limit is soft - the
server watches itself:

* Every 5 seconds it reads its memory (Linux: the resident memory, Windows: the working set). The console's `status`
  shows it: `RAM 600 MB (Grenze 1024 MB, Spitze 610 MB)` (limit, highest so far).
* At the start the log shows `RAM-Grenze: 1024 MB (der leere Server belegt 485 MB).` (the empty server's use). If the
  limit leaves too little room for missions, it says so: `Warnung: Für Einsätze ist die RAM-Grenze knapp – besser
  mindestens 1024 MB.` (the limit is tight, better at least …).
* From 90 % of the limit, a running mission keeps only 100 objects on the server's floor (spent cases, magazines,
  shards, rubble - nobody sees the server's floor; the players keep theirs).
* Above the limit the log warns: `Warnung: RAM-Grenze überschritten – 1100 von 1024 MB.` (limit exceeded). The server
  keeps running.

An empty server needs about 400 MB (the exported build), a mission more, depending on the map and the enemies. The
memory stays near the largest mission played until the server stops; the sleep mode frees all of it. For a **hard**
limit, run the instance in a Docker container and set AMP's container memory limit (edit the instance in AMP's
instance list): at that limit the system ends the server.

## 8. Playing

In the game: **Multiplayer → Beitreten** (join), enter the server's `address:port` as the lobby code (e.g.
`203.0.113.7:47250` or `swat.example.com:47250`) and the password. The server's name is shown at the top of the lobby.

* **Lobby leader:** whoever has been in the lobby the longest is the lobby leader (marked "Leiter" in the list). The
  leader picks map and mission, can remove players, and after a mission chooses between restart, next map and lobby.
* **Start:** once more than half of the players are ready, the mission starts after the countdown.
* **Version:** the server and the players need **the same game version**; otherwise the server refuses with "Andere
  Spielversion" (different game version).

## 9. Console

The commands work in English and in German; the answers are German.

| Command | Effect |
|---|---|
| `help` | all commands |
| `status` | state, frame rate, RAM, settings, players |
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

## 10. New game version

1. Put the new server archive at the same download address, or upload it.
2. Click **Update**.
3. Restart the server.

The players need the same game version as the server.

## 11. New template version

When this repository changed (see `CHANGELOG.md`):

1. In AMP (the ADS instance): **Configuration → Instance Deployment → Configuration Repositories → Fetch**.
2. In the instance list, click the **Update** button on the Swatted instance's tile (older AMP versions: right-click
   the instance → **Refresh Configuration**). Only this takes the new template; "Update" inside the instance does not.
3. Open the instance, check the new settings, and restart the server.

**AMP 2.8: "Config manifest file does not exist: Plugins/ADSModule/DeploymentTemplates/--main/swattedconfig.json".**
If the tile's **Update** ends with this error (ADS's notifications), AMP could not apply the new template: the instance
keeps its old version (the new settings do not show), and it can hang in "Stopping", because AMP stops the server for
the upgrade. This looks like a bug of AMP 2.8 with template repositories of one's own (an empty user and repository in
the path; another one has been reported in CubeCoders' forum). The way round it is a new instance, which gets the
current template: stop the old instance from ADS, create a new Swatted instance (step 3), enter the settings and the
port again, upload the server archive or click **Update** in the instance (step 4), start it, then delete the old one.

## Troubleshooting

* **"Keine Verbindung zum Host"** (no connection to the host) when joining: the UDP port is not open (firewall,
  router), or the address or port is wrong. AMP shows the port under Network, the console in the line `Server bereit`.
* **AMP stays at "Starting":** look at the console. "kein freier UDP-Port" (no free UDP port) means the port is taken.
  If the program is missing, Update ran without an archive (check the download address, or upload it by hand).
* **Linux: "Permission denied":** click **Update** once; it sets the execute permission.
* **AMP stays at "Stopping":** the console shows `Server wird beendet.` but AMP does not finish. On AMP 2.8 this
  came with a template upgrade that had failed (step 11, the "--main" error): stop or restart the whole instance from
  ADS, and if the error comes again, use a new instance.
* **The first join after a while fails:** the server was asleep. Join again after a few seconds, or update the game:
  newer versions wait for the server to wake up.
* **The server sleeps while someone plays:** AMP counted nobody. Check that the console shows the `Beigetreten` lines;
  with a server build from before the player ID, two players of the same name count as one - update the server.
