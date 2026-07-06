# LECmd Wrapper

A single-file **HTA GUI** for triaging Windows shortcut (`.lnk`) files with Eric Zimmerman's
[**LECmd**](https://github.com/EricZimmerman/LECmd). It runs LECmd for you and turns the CSV into an
interactive, suspicion-scored DFIR view built for the cases that actually matter: **weaponized / phishing
shortcuts** (ClickFix-style droppers), **USB / removable-media usage**, and **foreign-machine origins**.

No install — double-click the `.hta` (it runs via `mshta.exe`). Part of a family of EZ-tool wrappers that
share a common look, self-update, and command-line contract.

## Features

- **Runs LECmd for you** — point at a single `.lnk`, a collected shortcut folder (KAPE / Velociraptor
  trees work as-is), or your own `Recent` folder. Or **Load existing CSV…** to skip processing.
- **Two views of one dataset**
  - **Links** — one row per shortcut: tags, target, arguments, first/last opened, drive, volume, machine.
  - **Timeline** — one row per activity event (first-opened / last-opened), sortable and date-filterable.
    Optionally fold in the target file's created/modified timestamps.
- **Suspicion scoring** tuned for real intrusions (score ≥ 2 shades the row):

  | Tag | Meaning |
  |---|---|
  | **WEAPON** | Arguments match a LOLBIN / download / encoded-command pattern — classic weaponized shortcut |
  | **ARGS** | Shortcut carries command-line arguments (shell-generated Recent shortcuts almost never do) |
  | **EXEC** | Target is an executable / script |
  | **USERPATH** | Target sits under a user-writable / staging path (AppData, Temp, Downloads, …) |
  | **DBLEXT** | Double extension (e.g. `Invoice.pdf.exe`) or right-to-left-override in the name |
  | **MASQ** | An OS-binary name used as a target **outside** `\Windows\` |
  | **IOC** | Matches your IOC / keyword list |
  | **BIGLNK** | The `.lnk` file itself is unusually large (can embed a payload) |
  | **NET / USB** | Network (UNC) / removable-media target — informational filters, not alarms |

- **Overview panes** — recently opened, a **Volumes & origins** pane (distinct volume serials/labels and
  machine IDs for USB + foreign-host reconstruction), and score-ranked suspicious shortcuts.
- **IOC / keyword list** — paste or load a file (e.g. a Hawk IOC list); records rescore live.
- **Filters** — full-text search, UTC date range, and category buttons with live counts.
- **Export** the filtered view or timeline to CSV, or **Copy for case notes**.
- **Detail pane** — every timestamp with its real meaning, all path reconstructions, full arguments with
  IOC highlighting, and the complete volume / tracker (MachineID, MAC + vendor) block.
- **Self-updating** — checks its own GitHub release on launch and offers a one-click update.

## Quick start

1. Put `LECmd-Wrapper.hta` next to `LECmd.exe`, or use **Update / download LECmd** to fetch it.
2. Click **Scan my Recent folder**, browse to a collected shortcut folder, or pick a single `.lnk`.
3. **Process → analyze.**

## Command line

```
mshta "LECmd-Wrapper.hta" "<inputOrCsv>" ["<outDir>"] [/auto]
```

- `<input>` — a `.csv` (auto-loads into the viewer) or a `.lnk` file / shortcut directory (prefilled;
  processed immediately with `/auto`).
- `<outDir>` — CSV output directory (optional; defaults to `_Processed\<host>\LECmd` next to the app).
- **Target hostname** is required before processing — it names the `_Processed\<host>\LECmd` output folder next to the app (family convention shared with the DFIR-Artifact-Finder, so processed evidence is visible per host per tool). Guessed from `Collection-<host>-…` paths, a passed `_Processed\<host>\` outDir, or this machine's name for live paths — overwrite the guess if it's wrong.
- **Shared IOC list** — an `IOC.txt` next to the app (one term per line, `#` comments) is auto-merged into the IOC box at launch; one list covers the whole toolkit and terms you paste locally are kept.
- **Run provenance + triage summary** — every successful run appends a `runinfo.json` entry (app, host, input path, files) in the output folder, including a triage summary (entries, flagged count, max score, top hits); the DFIR-Artifact-Finder shows these per host in its inventory, even for standalone runs.

This is the shared contract used by the DFIR-Artifact-Finder launcher.

## Notes & limitations

- **All timestamps are UTC.** `First opened` = lnk created (target first opened); `Last opened` = lnk last
  written (most recent open). `Source accessed` is **unreliable** — the collection/parse itself updates it.
- The `FileSize` LECmd reports is the **target's** size at link time, not the `.lnk` size; the `.lnk` size is
  read separately (and only when the source file is reachable from this machine).
- LECmd (like several EZ tools) needs a **real console window** — this wrapper always runs it in a visible
  console; version checks use `--version`, which doesn't.
- Reading another user's profile or a protected folder needs an **elevated** run.
- Requires Windows with `mshta.exe` (built in) and .NET for LECmd (the download uses the .NET 9 build; the
  wrapper sets `DOTNET_ROLL_FORWARD=Major`).

## License

MIT © 2026 Ben Morris. LECmd is © Eric Zimmerman and distributed separately.
