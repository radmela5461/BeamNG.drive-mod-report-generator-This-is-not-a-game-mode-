BeamNG Mod Risk Report — Detailed Explanation (Open Source)
1) What this tool is

BeamNG Mod Risk Report is a local, offline analysis tool written in Python that inspects BeamNG.drive mod archives (.zip files) and generates a human-readable TXT report.

It is designed to help you answer questions like:

“Which mods are most likely to cause performance drops (FPS loss, stutter, long loading)?”

“Which mods are most likely to break textures/materials?”

“Which mods conflict in critical areas (Lua/UI/core overrides) and may cause instability?”

“Are there antivirus detections inside the folder I am scanning (Microsoft Defender on Windows)?”

This tool does not modify BeamNG.drive, does not install mods, does not delete files, and does not upload your mods anywhere.

2) Language and platform

Programming language: Python 3

Runs on: Windows / Linux / macOS (core analysis)

Antivirus scan feature: Windows only (uses Microsoft Defender via PowerShell)

If you run it on Linux/macOS, the mod analysis still works, but the Defender scan is automatically skipped.

3) Why this exists (the problem it solves)

BeamNG.drive mods are usually distributed as ZIP archives. Many mods can coexist peacefully, but some cause:

Performance issues: heavy textures, huge meshes, excessive scripts, massive file counts

Visual issues: broken textures, incorrect materials, pink/white surfaces, missing textures

Conflicts: one mod silently overrides files from another mod, especially in critical directories

This tool analyzes mod ZIP contents without extracting them, identifies suspicious patterns, and produces a report you can share with others or use for troubleshooting.

4) What this tool does (high-level)

When you run the script, it performs these actions in order:

Creates required folders (if missing):

mod test

long/log

Reads every .zip file inside mod test

Indexes each ZIP archive’s internal file list into a local SQLite database (stored locally)

Calculates “risk scores” for mods based on:

total size

texture size

mesh size

Lua/script size

file count

presence of special directories known to be conflict-prone

Detects critical conflicts between mods (only in important override areas)

Detects texture/material break candidates (mods likely to cause visual issues)

On Windows: optionally performs a Microsoft Defender scan of the mod test folder

Writes a detailed TXT report to:

long/log/beamng_report_YYYY-MM-DD_HH-MM-SS.txt

5) What the tool DOES NOT do (important trust section)

This tool:

Does not download anything

Does not connect to the internet

Does not send your mods, file names, or any data to any server

Does not install or remove mods

Does not change your BeamNG files

Does not modify ZIP archives

Does not run any code inside the mods

It only reads file names and sizes from ZIP metadata and uses those to compute risk signals.

6) How the analysis works internally (detailed)
6.1 ZIP inspection (no extraction required)

BeamNG mods come as .zip. This tool uses Python’s built-in ZIP support to read:

internal file paths (example: vehicles/carname/carname.jbeam)

internal file sizes

No extraction is required. That means:

no temporary mod files are created

no mod content is executed

the scan is fast and safe

6.2 Normalization (how file paths are compared)

Operating systems can vary in path formatting. The tool normalizes every ZIP entry:

converts backslashes to forward slashes

removes leading slashes

converts to lowercase

This increases reliability when comparing conflicts across mods.

6.3 Local SQLite indexing (why it exists)

Instead of storing everything in RAM (which can crash with hundreds/thousands of mods), the tool stores indexed entries in a local SQLite database:

file: long/perf_index.db

this database is created fresh each run

SQLite is used because it is:

local-only

reliable

fast

included with Python by default (no extra install)

6.4 Risk scoring (how “Top Risk Mods” are calculated)

The tool computes a numeric score per mod using multiple signals:

Total ZIP content size (bigger mods can load slower)

Texture size (.dds, .png, .jpg, etc.)

large texture sets often cause VRAM pressure and stutter

Mesh size (.dae, .fbx, .gltf, etc.)

large geometry can affect GPU load and memory usage

Lua/script size (.lua)

scripts can increase CPU usage or cause errors

File count

huge file counts can slow down scanning/loading phases

“Hotspot directory bonuses”

presence of directories that commonly cause conflicts:

core/

ui/

lua/

scripts/

vehicles/common/

mods touching these areas have higher risk because they override common/shared game logic

Important note:

This is a heuristic risk ranking. It does not guarantee a mod is bad.

It is a practical signal to help you narrow down candidates quickly.

6.5 Critical conflict detection (what is a “critical conflict”?)

Many mods “conflict” in harmless ways (example: two mods include their own unrelated files).
This tool focuses on conflicts that are more likely to cause actual issues.

A critical conflict is detected when two different mods contain the same internal file path inside these areas:

core/

ui/

lua/

scripts/

vehicles/common/

material definition files like materials.cs / materials.json

If two mods include the exact same file path in those areas, one can override the other depending on load order, causing:

instability

broken UI/apps

script errors

unexpected behavior

sometimes performance problems due to repeated script hooks or broken assets

The report lists these mod pairs under Critical Conflict Pairs.

6.6 Texture/material break detection (what it checks)

“Texture breaking” in BeamNG is commonly caused by:

material definition overrides (materials.cs, materials.json, or files inside materials/)

collisions on shared texture paths (same .dds path used by multiple mods)

This tool flags mods as “Texture/Material Break Risk” if they contain material definition files.
These are the most frequent root cause of:

pink/white textures

missing textures

wrong reflections

incorrect shaders/material properties

The report lists these mods under Texture/Material Break Risk Mods.

7) Antivirus scanning (how it works, and what it means)

7.1 Windows only (Microsoft Defender)

On Windows systems, the tool can run a Microsoft Defender Custom Scan on the mod test folder using PowerShell.

It does not install an antivirus. It uses what Windows already provides.

7.2 What gets reported

In the TXT report you’ll see:

AV Scan Results

Status: ok/error/unsupported

Detections: N

For each detection:

Threat name

Detection time

File path(s) reported by Defender

7.3 If Defender is disabled or blocked

Some systems may block Defender commands or PowerShell policies.

In that case:

the script still completes

the report shows Status: error and prints the error text (if available)

8) What the TXT report contains

Every generated report is saved to:

long/log/beamng_report_YYYY-MM-DD_HH-MM-SS.txt

The report includes:

General summary

total scanned zips

skipped/bad zips

total number of critical conflict pairs

AV Scan Results

whether antivirus scan ran

number of detections

details (if any)

Top Risk Mods

ranked list of mods with a risk score

includes approximate sizes (total/tex/mesh/lua) and file counts

Critical Conflict Pairs

mod A <-> mod B

number of critical overlapping file paths

Texture/Material Break Risk Mods

mods likely to affect materials/textures

9) What the user must do (step-by-step)
Requirements

Python 3 installed

Setup

Put the script file in a folder you want to use.

Run it once (it will create folders automatically).

Copy BeamNG mod .zip files into:

mod test

Run

On Windows:

Double-click the .py file

or run from CMD:

python beamng_mod_risk_report.py

On Linux/macOS:

python3 beamng_mod_risk_report.py

Output

After it finishes, open:

long/log/beamng_report_*.txt

The console stays open until you press Enter, so you can see the progress steps.

10) Privacy and security

This tool is designed for trust:

It runs locally.

It does not call any web APIs.

It does not upload your mods.

It reads ZIP metadata (names/sizes) only.

Antivirus scanning uses Windows Defender on your machine.

11) Open Source statement (very important)

This project is 100% open source.

That means:

You can inspect the entire code.

You can modify it.

You can build your own version.

You can verify what it does and does not do.

There are no hidden components, no remote services, and no obfuscated binaries.

If you are distributing this tool, it is recommended to distribute:

the .py source file

this README text

optionally a hash (SHA256) so others can verify the file has not been modified

12) Practical interpretation tips

A mod appearing in “Top Risk Mods” does not automatically mean it is “bad”.
It means it is “heavy” or “conflict-prone”, so it is a strong candidate if you have FPS drops.

If your game has visual glitches:
check “Texture/Material Break Risk Mods” first.

If your game has UI/app/script issues:
check “Critical Conflict Pairs” first.

If you want, I can also write a short version of this README for people who hate long documents—while keeping this detailed one for those who want full transparency.
