# parse.net

Standalone .NET replay parser and exporter for Fortnite replays.

This project was originally forked from [Shiqan/FortniteReplayDecompressor](https://github.com/Shiqan/FortniteReplayDecompressor). The original parser and replay model code still come from that codebase. On top of it, this fork adds a text/JSON export pipeline and a small console runner that writes analysis artifacts for each replay.

## What this fork adds

- JSON export for each parsed replay
- Human-readable TXT analysis output
- Owner and ranking analysis
- Kill feed summary and replay stats
- Output written to `REPLAYS/NET`
- Optional long-running replay watcher (`ReplayWatcher`) for automatic parsing after a match finishes

## Requirements

- .NET SDK 10.0 or newer
- Windows PowerShell or a terminal capable of running `dotnet`

If you need to install the SDK on Windows, the quickest path is:

```powershell
winget install --id Microsoft.DotNet.SDK.10 --exact
```

## Install

From the `parse.net` folder:

```powershell
dotnet restore
dotnet build -c Release
```

## Run on replays

From the `parse.net` folder:

Parse a single replay file:

```powershell
dotnet run --project src/ConsoleReader/ConsoleReader.csproj -c Release -- ..\REPLAYS\YourReplay.replay
```

Parse a folder of replay files:

```powershell
dotnet run --project src/ConsoleReader/ConsoleReader.csproj -c Release -- ..\REPLAYS
```

From the repository root (alternative):

Parse a single replay file:

```powershell
dotnet run --project .\parse.net\src\ConsoleReader\ConsoleReader.csproj -c Release -- .\REPLAYS\YourReplay.replay
```

Parse a folder of replay files:

```powershell
dotnet run --project .\parse.net\src\ConsoleReader\ConsoleReader.csproj -c Release -- .\REPLAYS
```

The exporter writes these files to the repository root under `REPLAYS/NET`:

- `<replay-name>.json`
- `<replay-name>.txt`

## Auto watch new replays (ReplayWatcher)

`ReplayWatcher` is a dedicated .NET console app that keeps running until you close it (or press `Ctrl+C`).
It scans for new `*.replay` files, waits until files are stable, then writes both JSON and TXT and prints the same text report to console.

Default watch directory:

- `%LOCALAPPDATA%\FortniteGame\Saved\Demos`

If the default replay directory does not exist yet, the watcher stays open and prints periodic waiting status messages.

From `parse.net` folder:

```powershell
dotnet run --project src/ReplayWatcher/ReplayWatcher.csproj -c Release
```

From repository root:

```powershell
dotnet run --project .\parse.net\src\ReplayWatcher\ReplayWatcher.csproj -c Release
```

Common options:

- `--dir <path>`: custom replay directory
- `--output-subdir <name>`: subfolder created inside watched replay directory (default: `ANALYSIS`)
- `--scan-interval <seconds>`: polling interval in seconds (default: `2`)
- `--process-existing`: also process already-existing replay files on startup

Examples:

```powershell
# Watch repository REPLAYS folder and write to REPLAYS\ANALYSIS
dotnet run --project .\parse.net\src\ReplayWatcher\ReplayWatcher.csproj -c Release -- --dir .\REPLAYS

# Process existing files too
dotnet run --project .\parse.net\src\ReplayWatcher\ReplayWatcher.csproj -c Release -- --dir .\REPLAYS --process-existing
```

## Build standalone EXE (no SDK/runtime required on target machine)

From repository root:

```powershell
& "C:\Program Files\dotnet\dotnet.exe" publish .\parse.net\src\ReplayWatcher\ReplayWatcher.csproj -c Release -r win-x64 --self-contained true /p:PublishSingleFile=true /p:IncludeNativeLibrariesForSelfExtract=true -o .\parse.net\dist\ReplayWatcher
```

Published executable:

- `parse.net\dist\ReplayWatcher\ReplayWatcher.exe`

Run it directly:

```powershell
.\parse.net\dist\ReplayWatcher\ReplayWatcher.exe
```

## Notes

- The parser logic itself is kept in the original .NET model/parser code.
- The console runner only adds reporting and export behavior.
- TXT output is formatted for scanability and is meant to stay stable over time.

## TESTRUN

The last verification run was executed with:

```powershell
& "C:\Program Files\dotnet\dotnet.exe" run --project .\parse.net\src\ConsoleReader\ConsoleReader.csproj -c Release -- .\REPLAYS\UnsavedReplay-2026.05.22-19.47.26.replay --quiet
```

## CHANGES

src/Program.cs

- Reduced to a thin entry point.
- Now only delegates to `ReplayAnalyzer.Run(args)`.
- Moved from `src/ConsoleReader/Program.cs` to `src/Program.cs`.

src/ReplayAnalyzer.cs

- Fixed `Final Kills` owner split output so `(Real Players / Bots/NPC)` is counted from final kills only (knocks excluded), matching the number before parentheses.
- Fixed summary header consistency: `Real Players` and `Bot/NPC` are now counted from ranked participants (same basis as survivors/ranking), so they align with `Total Players` instead of raw `PlayerData Records`.
- Formatted summary header block labels/values into an aligned column so results are visually under each other.
- Shortened summary labels to save space: removed `(AthenaMatchTeamStats)`, changed `(IsBot=false)` to `(!isBot)`, and `(IsBot=true)` to `(isBot)`.
- Tightened summary value-column spacing (reduced label width) so results do not start too far to the right.
- Shifted the summary value column an additional 8 characters to the left on request.
- Refined summary spacing to a balanced, clean alignment so all header values remain clearly in one column.
- Renamed owner `Final Kills` to `Kills` and aligned it with `PlayerData.Kills` (same source as ranking).
- Added explicit `Kill Feed: X killed / Y knocked` line to make kill-feed vs ranking differences transparent and consistent.
- Added reusable `ProcessReplayFile(string replayFilePath, string outputRoot)` API for external callers (for watcher mode).
- `WriteArtifacts(...)` now returns generated artifact paths and content via `ReplayProcessResult`.

src/ConsoleReader/ConsoleReader.csproj

- Added compile include/link for `../ReplayAnalyzer.cs` so the analyzer code can live in `src` root and still build with ConsoleReader.
- Added compile include/link for `../Program.cs` after moving the entry point to `src` root.
- Removed hardcoded `Replays/*.replay` `None Update` entries (no forced test replay copy to output anymore).

README.md

- Removed stale Node.js-specific wording from the .NET project readme.
- Added fork-origin explanation (based on original Shiqan/FortniteReplayDecompressor codebase).
- Added .NET installation and usage instructions.
- Added test-run command section with exact executed command.
- Added and maintained `CHANGES` / `ADDED` tracking section.
- Clarified run commands for both working directories (`parse.net` and repository root).
- Updated TESTRUN command to the latest verified single-replay run.
- Added dedicated ReplayWatcher documentation (usage, options, behavior, and standalone EXE publish).

src/ReplayWatcher/Program.cs

- New long-running watcher entry point that polls for finished `*.replay` files.
- Uses file signature stability checks (size + last-write ticks) before processing.
- Writes JSON/TXT through `ReplayAnalyzer.ProcessReplayFile(...)` and prints TXT analysis to console.
- Keeps console open with status output and graceful `Ctrl+C` shutdown.
- Waits and retries when the default Fortnite replay folder does not exist yet.

src/ReplayWatcher/ReplayWatcher.csproj

- New watcher project that links `../ReplayAnalyzer.cs` and references parser dependencies.
- Can be published as self-contained single-file Windows executable.

## ADDED

src/ReplayAnalyzer.cs

- New standalone analyzer/orchestrator file in `parse.net/src`.
- Contains all extended behavior that was previously in `ConsoleReader/Program.cs`:
- CLI option parsing (`--quiet`, `--output-root`, replay input)
- Replay batch execution and timing
- JSON and TXT artifact writing
- Formatted replay analysis generation
- Owner diagnostics output
- Workspace-root REPLAYS path resolution

src/ReplayWatcher/Program.cs

- Added permanent replay watcher executable flow for automatic post-match parsing.
- Added CLI flags: `--dir`, `--output-subdir`, `--scan-interval`, `--process-existing`.

src/ReplayWatcher/ReplayWatcher.csproj

- Added dedicated watcher project with linked analyzer source and required package references.


