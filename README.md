# winget-forensics

Central winget-pkgs update bot for kalink0's tools (crush, peach, ...). Holds
no manifests itself — those live upstream in `microsoft/winget-pkgs` — this
repo only automates keeping them current.

## How it works

`.github/workflows/bump.yml` runs every 6h (and on manual dispatch):

1. Reads `packages.json` for the list of tracked tools (winget identifier,
   source repo, Windows release-asset name pattern).
2. For each, checks the source repo's latest GitHub release tag against the
   version last recorded in `state.json`.
3. On a new version, runs `wingetcreate update <identifier> --submit`, which
   opens a PR against `microsoft/winget-pkgs`, then records the version in
   `state.json` (committed back) so the next run doesn't resubmit it.

A package that fails (most commonly: its identifier doesn't exist upstream
yet) is skipped for that run, not fatal — `state.json` stays unchanged and
it's retried on the next schedule.

Before any of that, the workflow syncs `kalink0/winget-pkgs` (our fork) with
upstream master via `gh repo sync`. Our fork only gets touched on actual
submits, so between infrequent releases it drifts hundreds/thousands of
commits behind — and both `wingetcreate` and `komac` fail with a misleading
permissions error (`does not have the correct permissions to execute
CreateRef`) when trying to branch off a fork that far out of sync, instead of
syncing it themselves. See
[microsoft/winget-create#580](https://github.com/microsoft/winget-create/issues/580)
and [russellbanks/komac#1142](https://github.com/russellbanks/Komac/issues/1142).
If you ever hit that error running either tool by hand (e.g. the manual
first-submission step below), sync the fork on GitHub (or `gh repo sync
kalink0/winget-pkgs -b master`) and retry.

## Adding a new tool

1. First submission to winget-pkgs has to happen once, by hand, with
   `wingetcreate new` (or `komac new`) — this is a human-reviewed process,
   deliberately not scripted here.
2. Add an entry to `packages.json`:
   ```json
   {
     "identifier": "kalink0.YourTool",
     "repo": "kalink0/your-tool-repo",
     "windows_asset_pattern": "your-tool-windows-v{version}.zip"
   }
   ```
3. Once the identifier exists upstream, the next scheduled run picks it up
   automatically.

## Secrets

- `WINGET_PAT`: classic GitHub PAT with `public_repo` scope, used by
  `wingetcreate` to open PRs against `microsoft/winget-pkgs`. Same PAT
  previously used directly in crush-forensics' own workflow.
