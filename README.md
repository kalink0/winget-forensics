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
