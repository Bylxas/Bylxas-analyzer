# Bylxas Analyzer

A PowerShell script that scans your Minecraft mods folder and flags cheat clients, sketchy mods, injection attempts, and obfuscated garbage hiding in your jars.

## What it actually does

Runs your mods folder through 5 passes:

**Pass 1 — Hash check.** Every jar gets SHA1'd and checked against Modrinth and a Megabase lookup. If it matches a known mod, it's verified and moves on.

**Pass 2 — Deep scan.** File names, paths, and class contents (both ASCII and UTF-8, so fullwidth Unicode tricks like ＡｕｔｏＣｒｙｓｔａｌ don't slip through) get checked against the pattern and string lists.

**Pass 3 — Bypass/injection scan.** Looks for `Runtime.exec()`, sneaky HTTP download or exfil code, suspicious nested jars, and mods pretending to be something they're not (fake mod IDs).

**Pass 4 — Obfuscation check.** Looks at class naming patterns — numeric names, single/double-letter names, no-vowel gibberish, fullwidth/Japanese characters — and checks for known obfuscators (Skidfuscator, Zelix, Paramorphism, etc).

**Pass 5 — JVM scan.** Checks the running java/javaw process for shady agents and flags like `-javaagent`, `-agentlib:jdwp`.

At the end you get a summary broken down into Verified / Unknown / Suspicious / Bypass-Injection / Obfuscated / JVM.

## Running it

```powershell
irm https://raw.githubusercontent.com/Bylxas/Bylxas-analyzer/main/Analyzer.ps1 | iex
```

or if you'd rather grab it first and look before running:

```powershell
irm https://raw.githubusercontent.com/Bylxas/Bylxas-analyzer/main/Analyzer.ps1 -OutFile Analyzer.ps1
.\Analyzer.ps1
```

It'll ask for your mods folder path, or just hit Enter to use the default (`%USERPROFILE%\AppData\Roaming\.minecraft\mods`).

## Adding new detections

Two lists do the heavy lifting:

- `$suspiciousPatterns` — matched against file paths, package names, class names
- `$cheatStrings` — matched inside the actual class bytecode (plus auto fullwidth-Unicode variants)

Editing a 600+ entry PowerShell array by hand is a pain, so there's a companion HTML editor that loads the script, lets you add/remove entries through a UI, and spits out the updated file.

## Worth knowing

This isn't an antivirus. It's pattern matching against known cheat clients and behaviors — it can miss brand-new stuff, and it can occasionally flag a legit mod with an unusual name. Treat a flag as "worth a closer look," not gospel.

---

Built on top of [MeowModAnalyzer](https://github.com/MeowTonynoh/MeowModAnalyzer) by [MeowTonynoh](https://github.com/MeowTonynoh).
