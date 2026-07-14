# Bylxas Analyzer

PowerShell script that scans your Minecraft mods folder and flags likely cheat clients.

## Installation

```
powershell -ExecutionPolicy Bypass -Command "Invoke-Expression (Invoke-RestMethod 'https://raw.githubusercontent.com/Bylxas/Bylxas-analyzer/main/Analyzer.ps1')"
```

## Usage

On startup you're asked for the mods folder path:

- Hit Enter for the default: `%USERPROFILE%\AppData\Roaming\.minecraft\mods`
- Or type in a custom path

## How It Works

### Pass 1: Hash Verification

Every jar gets SHA1'd and checked against:

**Modrinth API** — `https://api.modrinth.com/v2/version_file/{hash}`
Main lookup, returns the mod's name and slug when it's a match.

**Megabase API** — `https://megabase.vercel.app/api/query?hash={hash}`
Fallback lookup for mods Modrinth doesn't know about.

Anything that matches gets marked **VERIFIED**.

### Pass 2: Deep Scan

For anything not verified, the script opens the jar (`System.IO.Compression.ZipFile`), including nested jars under `META-INF/jars/`, and checks:

- File and folder names inside the archive
- Contents of `.class`, `.json`, and `MANIFEST.MF` files, read as both ASCII and UTF-8 (so fullwidth Unicode tricks like ＡｕｔｏＣｒｙｓｔａｌ get caught too)

Everything gets matched against the pattern and string lists below.

### Pass 3: Bypass / Injection Scan

Digs into class bytecode looking for:

- `Runtime.exec()` calls combined with heavy obfuscation
- HTTP download or exfiltration behavior (`openConnection` + file writes / POST requests)
- Suspicious nested jars with no version number or unfamiliar naming
- A mod claiming a known-legit mod ID while shipping dangerous code

### Pass 4: Obfuscation Check

Looks purely at how classes are named and structured:

- Numeric-only names, single/double-letter names
- No-vowel or consonant-cluster gibberish names
- Fullwidth or Japanese characters in class names
- Single-character package path segments
- Known obfuscator fingerprints (Skidfuscator, Zelix, Paramorphism, Radon, Caesium, and others)

### Pass 5: JVM Scan

If Minecraft is running, checks the live java/javaw process for:

- Unrecognized `-javaagent` attachments
- Risky JVM flags: `-Xbootclasspath/p:`, `-Xbootclasspath/a:`, `-agentlib:jdwp`, `-agentpath:`

### Download Source Tracking

Reads the Windows `Zone.Identifier` alternate data stream to figure out where a jar was actually downloaded from.

**Usually fine:** CurseForge, Modrinth

**Worth a second look:** Discord/Discord CDN, MediaFire, GitHub, MEGA, Dropbox, Google Drive, AnyDesk, and a few known cheat-distribution domains

## Detected Patterns

Several hundred entries across two lists — a sample of what's covered:

**Combat:** `AimAssist`, `AutoCrystal`, `AutoHitCrystal`, `TriggerBot`, `KillAura`, `Criticals`, `ReachHack`, `ShieldBreaker`, `ShieldDisabler`, `AxeSpam`

**Movement:** `FlyHack`, `Antiknockback`, `NoKnockback`, `JumpReset`, `SprintReset`, `NoJumpDelay`, `BHop`

**PvP utility:** `AutoTotem`, `AutoArmor`, `AutoPot`, `AutoDoubleHand`, `InventoryTotem`, `PopSwitch`, `LagReach`, `FakeLag`

**Visual/spoofing:** `BlockESP`, `Freecam`, `PackSpoof`, `PingSpoof`, `Fakenick`, `PlayerESP`, `Tracers`

**Automation:** `FastPlace`, `ChestSteal`, `AutoEat`, `AutoMine`, `AutoClicker`, `FastXP`

**Known clients:** `meteordevelopment`, `liquidbounce`, `impactclient`, `Asteria`, `Prestige`, `Xenon`, `Argon`, `Hellion`, `Grim`, `catlean`, `dev.krypton`

**Obfuscated/hidden:**
- Package `org.chainlibs.module.impl.modules.*`
- Classes named with hiragana/katakana (`じ.class`, `ふ.class`, etc.)
- Suspicious mixins: `LicenseCheckMixin`, `ClientPlayerInteractionManagerAccessor`
- Marker files: `phantom-refmap.json`, `client-refmap.json`
- Native input libs: `jnativehook`, `imgui`, `imgui.gl3`

...and quite a bit more.

## Output

Results are grouped into:

**VERIFIED MODS** — matched a known database entry.

**UNKNOWN MODS** — no database match, but nothing suspicious found either. Download source shown if available.

**SUSPICIOUS MODS** — matched one or more cheat patterns/strings, listed per file.

**BYPASS / INJECTION** — flagged for exec calls, network exfiltration, or identity spoofing.

**OBFUSCATED MODS** — flagged purely on class-naming/structure heuristics.

**JVM** — flagged agents or flags on the running Minecraft process.

## Extra Info

If Minecraft is currently running, the script also shows the process name, PID, start time, and uptime.

## Editing the detection lists

`$suspiciousPatterns` and `$cheatStrings` are plain PowerShell arrays, but with 700+ combined entries, hand-editing gets tedious fast. There's a companion HTML tool that loads the script, lets you add/remove entries through a simple UI, and exports the updated file.

## Contacts

Discord: `Bylxas187`
GitHub: [Bylxas](https://github.com/Bylxas)

---

Built on top of [MeowModAnalyzer](https://github.com/MeowTonynoh/MeowModAnalyzer) by [MeowTonynoh](https://github.com/MeowTonynoh).
