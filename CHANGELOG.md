# Changelog

The AMP template's versions. Each one raises `Meta.ConfigVersion` in `swatted.kvp`; after the merge, AMP takes it with
**Fetch** and the instance's **Update** (README, step 11).

## [5] – 2026-11-14

Best with a Swatted server build from 0.11.0 on that names its memory manager in the log (the branch
`worktree-feat-server-ram-saver` of the game on); an older build runs with jemalloc too, it only does not say so.

### Added

- **jemalloc on Linux:** AMP installs `libjemalloc2` into the container (`Meta.ExtraContainerPackages`), the start
  preloads it (`App.EnvironmentVariables`: `LD_PRELOAD`, `MALLOC_CONF` with a background thread that gives freed memory
  back after a second; `MALLOC_ARENA_MAX` 2 for glibc where jemalloc is missing). Measured with the exported build:
  the empty server 191 instead of 375 MB, a mission 353 instead of 403 MB, back in the lobby 313 instead of 403 MB.

### Changed

- README step 7: jemalloc, the measurements, how to check it, the hard limit (512 MB), why the frame rate stays;
  the troubleshooting for `Speicherverwaltung: glibc`. The setting "Maximaler RAM" names the new figures.

## [4] – 2026-11-14

### Changed

- **Sleep mode on** again, as in version 2 (`App.SupportsUniversalSleep=True`,
  `App.UniversalSleepApplicationUDPPortRef=GamePort`, `Limits.SleepMode=True`). CubeCoders switched sleep off in their
  own templates (October 2026); a template from another repository keeps its own setting. Use AMP 2.8.0.10 or newer.
- README: sleep mode again; the AMP 2.8 upgrade error ("--main") and its way round (a new instance) in step 11 and the
  troubleshooting.

### Fixed

- The reason in version 3 was wrong: sleep mode did not make AMP hang in "Stopping". The instance never got version 2:
  AMP's template upgrade failed ("Config manifest file does not exist:
  Plugins/ADSModule/DeploymentTemplates/--main/swattedconfig.json"), and the hang came with that failed upgrade.

## [3] – 2026-11-14

### Changed

- **Sleep mode off** again (`App.SupportsUniversalSleep=False`, `Limits.SleepMode=False`): AMP 2.8 has switched sleep
  mode off for all Generic-module templates for now. (A second reason given here, that sleep mode made AMP hang in
  "Stopping", was wrong - see [4].)
  The player IDs in the join / leave lines and the RAM limit stay.
- README: the sleep mode section says why; troubleshooting for "Stopping".

## [2] – 2026-11-14

Needs a Swatted server build that knows `--max-ram` (0.11.0 from the branch `feat/server-ram-sleep` on). An older
build logs "Unbekannte Option „max-ram“" and otherwise runs as before.

### Added

- **Maximaler RAM** (MB, 0 = no limit), passed as `--max-ram`: the server watches its memory against it (soft limit;
  the hard one is AMP's container memory limit).
- **Sleep mode**, on by default: with nobody playing for 5 minutes AMP stops the server and listens on its UDP port;
  the first player who joins wakes it.
- `CHANGELOG.md`.

### Changed

- The join and leave regexes read the player's ID (`, ID <id>` at the end of the server's lines), so two players of
  the same name count as two; lines without it still match.
- README: sleep mode, RAM limit, updating the template in AMP, troubleshooting; this repository is the template's only
  copy.

## [1] – 2026-11-14

- The first template: Swatted's dedicated server (`--headless -- --server`) in AMP's Generic module - settings,
  UDP port 47250, download from an address or by hand, console, the ready / join / leave lines.
