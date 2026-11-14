# Changelog

The AMP template's versions. Each one raises `Meta.ConfigVersion` in `swatted.kvp`; after the merge, AMP takes it with
**Fetch** and the instance's **Update** (README, step 11).

## [3] – 2026-11-14

### Changed

- **Sleep mode off** again (`App.SupportsUniversalSleep=False`, `Limits.SleepMode=False`): AMP 2.8 has switched sleep
  mode off for all Generic-module templates for now, and with it on AMP hung in "Stopping" after the server had ended.
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
