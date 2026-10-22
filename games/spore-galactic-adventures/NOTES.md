# Spore: Galactic Adventures — co-op notes

Working notes for the Spore: Galactic Adventures co-op effort. Written as I go.
Sources are linked; observations, derivations, guesses and unknowns are kept apart.

Game identity: see `local/identity.md` (private, git-ignored).
Phases finished: **0 Triage**, **1 Research**, **2 Workspace**, **3 Understand**,
**4 Choose**, **6 QA** (partial). Not started: **5 Build** (no native changes were
possible here), **7 Ship**, **8 Give back** (this file + `README.md` + the directory
entry are the give-back).

---

## Phase 0 — Triage

### 1. Which exact game?

Spore: Galactic Adventures (2009 expansion of Spore, 2008), Steam app **24720**,
Russian language, Windows. The Galactic Adventures executable is
`SporebinEP1\SporeApp.exe`; the base game executable is `SporeBin\SporeApp.exe`.
Both are present in the Steam depot and the GA exe matches one of the sizes the
existing mod explicitly supports (25 066 744 bytes).

> **Correction (2026-10-21, Phase 3).** The upstream script calls that size the
> "Steam March 2017" build. The ModAPI Launcher's own version table
> (`ModAPI.Common.GameVersion.ExecutableSizes`) maps 25 066 744 bytes to
> **`Steam_October2024`**. `Steam_March2017` is 24 885 248 bytes, a *different* exe.
> Both variants resolve to the same ModAPI core DLL (`SporeModAPI.march2017.dll`),
> so the mislabel is harmless for the kit — but the mod's own Detours hooks are
> pinned to hashed RVAs, and the october build is a different binary, so this
> matters for the native side.

The install was **not finished** on the owner's PC at triage time: `common\spore`
was empty and the payload was still in Steam's staging area.

### 2. Is it already done?

**Yes — a co-op mod exists**, but it stops at the Cell stage and was never
confirmed in a running game.

`emerald194520350-alt/spore-multiplayer-mod` — "Spore Multiplayer Mod", MIT,
29 stars, 21 commits between 2026-09-17 and 2026-09-27, latest release **v1.0.1**
(2026-09-27). Repository has **0 issues** and **0 open PRs**, so there is no public
bug list to mine — the author did the work in ten days and left no written
backlog. Every release note says the same thing in different words: *not verified
in a running game*.

There is **no entry for Spore** in this repo's `data/directory.yaml` and no
`games/spore/` folder, so nobody has covered this game here yet.

### 3. Which engine?

Proprietary EA/Maxis Spore engine with RenderWare rendering. No shipped
multiplayer, no netcode, no scripting layer. The community's answer to "closed
binary" is the **Spore ModAPI SDK** (`Spore-Community/Spore-ModAPI`, 86 stars),
a reverse-engineered C++ SDK that hooks the game with Microsoft Detours and ships
`SporeModAPI.march2017.dll` for exactly the March 2017 executable the owner has.
Mods are injected by the **SPORE ModAPI Launcher Kit**
(`Spore-Community/ModAPI-Launcher-Kit`, latest v1.5.13, 2026-10-16).

That puts Spore in the "plugin / script extender" tier of `playbook/triage.md`:
hook the engine from a DLL. No decompilation is needed for the mod itself, and
the ModAPI SDK ships Ghidra import files (`SporeGhidra_march2017.xml`) for anyone who
does need to go deeper.

### 4. How much source is there?

- The game itself: closed binary only.
- The mod: full C++ source (native client) + C# source (session server) + tests,
  MIT licensed, and the Spore ModAPI SDK vendored under `vendor/Spore-ModAPI`.
- The ModAPI SDK itself is public source (C++).

### 5. Does the engine have any multiplayer?

**No.** Spore has no multiplayer mode. The existing mod therefore had to build the
networking layer from scratch — and it did: a native client DLL talking to a
separate **C# TCP session server** (`SporeCoop.Server.exe`, `server/SessionServer.cs`).

### 6. Shipping check

Fine. The mod ships as a DLL + a launcher script; no game files, no assets, no
anti-cheat, no online service. Spore is an offline game. Nothing in the repo
touches EA or Valve services. The mod is MIT; the vendored ModAPI SDK and Detours
keep their own licences and the release archive ships the notices.

---

## Phase 1 — Research

### Searches run (2026-10-21)

| Where | Query | Result |
|---|---|---|
| GitHub repo search | `spore galactic adventures` | 32 repos; 1 co-op project (the mod below) |
| GitHub repo search | `spore modapi OR sporemods` | the ModAPI SDK and the community mod repos |
| GitHub repo search | `spore multiplayer` | see below |
| GitHub issues/PRs | `repo:cranebeaker/coop-anything spore` | **0 results** — nobody has touched Spore here |
| GitHub releases API | mod repo, 15 releases (beta-1 … v1.0.1) | read in full |
| GitHub issues API | mod repo | **0 issues, open or closed** |
| GitHub commits API | mod repo | 21 commits, all 2026-09-17 → 2026-09-27 |
| Nexus Mods | `spore` | **403 / Cloudflare challenge**, not readable from here |
| ModDB | `spore` multiplayer | nothing co-op for GA |
| Reddit | `spore multiplayer mod` | only the "Project Multispore" turn-based idea, no code |
| Steam forums (app 24720) | co-op | same "Project Multispore" thread, turn-based, no code |
| DavoOnline (via Wayback, 2024-06-24 snapshot) | Spore multiplayer | a 2017 thread with **concepts only**, no implementation |
| playground.ru, Discord | Spore co-op | no implementation found |
| coopmeld.com/spore | "Spore Coop" | **the most advanced project — see below** |

### The existing project

| Field | Value |
|---|---|
| Repo | https://github.com/emerald194520350-alt/spore-multiplayer-mod |
| Author handle | `emerald194520350-alt` (single author, no org) |
| Licence | MIT (`LICENSE` in repo) |
| Latest release | **v1.0.1**, 2026-09-27, protocol 10 |
| Release asset | `spore-multiplayer-mod-v1.0.1-windows.zip`, 357 314 bytes |
| Published SHA-256 | `787cac1f7b262dba76093923a75dcecc07916c4a5185182b94893b8c4d181e78` |
| **Verified SHA-256** | `787CAC1F7B262DBA76093923A75DCECC07916C4A5185182B94893B8C4D181E78` — **matches** |
| Downloads | 8 |
| Activity | 21 commits, all within 10 days, then silent |
| Issues / PRs | none |

The archive contains a prebuilt `bin\SporeCoop.Probe.dll` (857 600 bytes) and
`SporeCoop.Server.exe` (70 656 bytes), so the mod can be **used** without a C++
toolchain. Rebuilding from source needs Visual Studio 2022 (MSVC v143 + Windows
SDK); the C# server rebuilds with the .NET Framework 4 `csc.exe` that ships with
Windows; the `.mjs` test files need Node.js 22+.

### Architecture (read from source)

```
SporeApp.exe (window 1, host)  ─┐
    native/Probe.cpp  (C++, Spore ModAPI + Detours hooks)
    native/CoopNet.cpp (client protocol, JSON over length-prefixed TCP)
                                ├──► SporeCoop.Server.exe  (C#, TcpListener)
SporeApp.exe (window 2, guest) ─┘      server/SessionServer.cs
                                          • central broker + state authority
                                          • broadcasts position / appearance /
                                            npcSnapshot / worldAction
                                          • holds progress, unlocks, DNA budget,
                                            evolution state, species name
                                          • roles identified by host/guest token
```

This matters a lot: **the clients already talk to a server over TCP**, so
remote play is not a networking rewrite. The server is a plain process that can
listen on any address (`--listen`, `--port`) and the clients take their address
from environment variables (`SPORE_COOP_SERVER`, `SPORE_COOP_PORT`,
`SPORE_COOP_TOKEN`, `SPORE_COOP_ROLE`, `SPORE_COOP_PROFILE`).

### What the mod actually syncs (from the v1.0.1 README and release notes)

Shared: player movement and appearance, NPC replication, shared growth/DNA/parts,
diet history, species name, the creature editor (shared model + shared DNA budget,
with revision merging), shared progress and unlocks, History timeline, one-sided
campaign saves (owner writes, guest is protected), pause, editor entry/exit.

**Not** supported: Creature, Tribal, Civilization and Space stages. After the Cell
stage the mod shows an end screen instead of the landfall transition.

### Honest QA level of the existing mod

`qa: none` at triage time. The author's own words in the v1.0.1 release body:

> Fragment drops and collection in two windows still need gameplay verification.
> The ending screen, saving, return to the menu, and earlier guest fixes are also
> not confirmed by automated tests alone.

> These checks do not constitute a full two-player playthrough. (README)

There are 142 server checks, 115 client checks and 154 isolated engine checks,
which is a lot of automation — but the engine checks run against a **private
executable image** (the game binary mapped into memory without starting it), not
against a running game. **No two-window playthrough has ever been confirmed.**

### Known gaps, ranked by how much they block "play with friends"

1. **Remote play is unfinished.** README: *"Remote LAN/VPN play and complete world
   transfer between computers are still unfinished."* The launch scripts hardcode
   `127.0.0.1`. Beta 1 notes: *"Cross-PC Radmin VPN play and complete world
   transfer are not validated."*
2. **Only the Cell stage.** Four of the five stages have no co-op at all.
3. **Never verified end to end.** Every release stops at "needs two-window
   verification".
4. **Hardcoded developer paths** in the scripts: `C:\Games\SPORE Collection`,
   `C:\ProgramData\SPORE ModAPI Launcher Kit` and `...\ Launcher Kit 2`.
5. **Two Launcher Kit installs are required** (one per profile/window), and a
   second profile is created at `%APPDATA%\SporeCoop2` with the host's saved
   campaign and `Documents\My Spore Creations` copied into it.

### Other Spore projects seen (context, not co-op)

`bisfranz/SporeNEXT` (mod manager, v2.0.9), `Liskomato/Spore-*` (many small
Spore ModAPI mods), `MADPSI/*`, `0KepOnline/SPOREMod_*`, `Spore-Community/*`.
None of them are co-op. `SporeNEXT` may be useful for installing `.package` mods
but the co-op mod is a DLL and needs the Launcher Kit, not SporeNEXT.

### The other four Spore co-op projects (all dead ends for the Space stage)

| Project | What it is | Why it does not solve Space-stage co-op |
|---|---|---|
| **CoopMeld "Spore Coop"** — https://coopmeld.com/spore/ | GPL sources (installer + native mod + launcher), 2–4 players, online rooms, Cell+Creature+Tribe | The **live service is paused**: `/api/check?nonce=…` returns **HTTP 503 "Release unavailable."**, and `/lobby` returns **426 `{"error": "Update SporeCoop in the launcher before playing."}`**. The site itself lists Civilization and Space as *"Coming later"*. It is the only project that ever targeted more than the Cell stage. |
| **raph559/SporeMP** — https://github.com/raph559/SporeMP | GPL-3.0, roadmap for all stages, native dedicated server | Its byte-level guard pins the **GOG** build (24 895 536 B, SHA-256 `dc04aee5…`) and **rejects the Steam build** this owner has. Also a prototype. |
| **Spore-Multiplayer/Spore-Multiplayer** | empty Visual Studio skeleton, 18 280 B | `dllmain.cpp` is the ModAPI template; `.gitmodules` points at `emd4600/Spore-ModAPI`; `Client/README.md` says "Files for the Client. TBA". No code. |
| **sunglocto/SporeMultiplayer** | 545-byte README | README only, no code, no releases. |
| **NeDether/SporeMultiplayer** | — | **404 / gone.** |

**Conclusion of the research phase:** *no existing project makes Space-stage co-op
work.* The best available candidates are CoopMeld (paused service, never reached
Space) and the emerald mod (Cell only, works locally once launched correctly).

### Derived (my reasoning, not claims by the sources)

- **Remote play is mostly plumbing, not a rewrite.** The transport already exists
  and the server already speaks "one host authority + one guest" over plain TCP.
  Running `SporeCoop.Server.exe` on a reachable machine and pointing the guest at
  it is the natural design. This is a *design hypothesis*, not a tested fact.
- **The real remote-play blocker is world transfer.** The guest's profile must
  contain a copy of the host's saved campaign (`Games\` Game0 files) and the
  creature assets under `Documents\My Spore Creations`; `BorrowedCampaigns.h`
  hides those borrowed copies from the guest's galaxy menu and
  `CampaignSync.h` blocks the guest from writing them. The launcher does this
  copy locally. Across machines it has to be done by hand or implemented.
- **Steam relay is probably the wrong tool here.** The client DLL would have to
  talk to the Steam Spacewar relay, which is a lot of native work for no gain
  over "run the session server on a VPS/LAN box". Worth revisiting only if the
  owner wants zero-setup joins.
- The mod uses `DetourAttach` on hashed RVA addresses (`VerifiedMotionCode`), so
  it is pinned to the exact executable it was built against. A different GA build
  breaks it. *(The owner's build is the october one; the mod still initialised on
  it — see Phase 6 — but the RVA pinning is a real fragility.)*

### Unknowns

- Whether the mod works at all in two live windows. Nobody has confirmed.
- Whether the guest's local campaign must match the host's exactly, or only
  needs *a* valid Cell-stage campaign (the shared state sync would then decide
  the rest).
- Whether the "prepared copy" mechanism can survive a profile that is not a
  byte-for-byte copy of the host's.
- Whether Spore's cell stage can be entered at all on a fresh profile without
  going through the galaxy menu first.
- Issue tracker for bugs: the repo has none. The author's e-mail is not published.
  **There is nowhere to file a bug report against this project.**

---

## Phase 2 — Workspace

### Toolchain (owner approved installs)

| Tool | State |
|---|---|
| Python 3.12.10 + `pyyaml` 6.0.3 | installed (winget) |
| Node.js 24.20.0 | installed (winget) |
| Git 2.55.0.5, GitHub CLI 2.102.1 | installed (winget) |
| .NET Framework 4 `csc.exe` | present in Windows, used to build the C# session server |
| SPORE ModAPI Launcher Kit **v1.5.13** | installed at `C:\ProgramData\SPORE ModAPI Launcher Kit` (WiX bundle, `/quiet /norestart`, exit 0) |
| Visual Studio / MSVC v143 + Windows SDK | **declined by the owner** → native DLLs cannot be rebuilt here |

The kit install did **not** modify the game install: no `SporeApp_ModAPIFix.exe`
was created and `steam_api.dll` is byte-identical in both installs
(87 288 B, SHA-256 prefix `7AC1090D725DCBD3`). That compatibility executable is
only produced for the EA App build; this install is Steam.

A **second copy** of the kit was made by copying the folder to
`C:\ProgramData\SPORE ModAPI Launcher Kit 2` (not a second WiX install). Phase 3
explains why two copies are genuinely required.

### Test copy of the game

Per the repo ground rules the owner's install and saves are never touched. A
**robocopy** clone lives in a separate test folder (exit code 1 = files copied;
1458 files, 6 151 060 957 B). Every launch test in this file points there.

The only file ever written into any Spore folder is
`<test copy>\SporebinEP1\steam_appid.txt` — see Phase 3, it is required
for the game to start outside Steam, and it was written to the **test copy only**.

---

## Phase 3 — Understand

This phase produced the single most useful result of the whole effort: the mod
does run in this game, but the upstream launch script cannot start it.

### 3.1 The game would not start (exit code 53)

**Observation.** Launching `<game>\SporebinEP1\SporeApp.exe` directly (not through
Steam) made the process exit immediately:

| `steam_appid.txt` in `SporebinEP1` | Result |
|---|---|
| absent | process exits, **code 53** |
| contains `17390` | process exits, **code 86** |
| contains `24720` | **runs normally** (≈ 408 MB working set, window present) |

**Root cause (derived, then confirmed).** The Steam build calls `SteamAPI_Init`,
which fails when the process was not started by the Steam client. `steam_api.dll`
reads the app id from `steam_appid.txt` in the game folder when the process is
not Steam-launched. The file has to contain the **app id of the executable that
is running**: the GA executable is app **24720** (the base `SporeBin` game is
app 17390). The value was confirmed independently from the ModAPI Launcher's own
constant `ModAPI.Common.SporePath.GalacticAdventuresSteamID = "24720"`.

The ModAPI Launcher **does this automatically** in its GUI flow, and the upstream
`Start-SporeCoopInstance.ps1` **bypasses that flow** (it calls
`InjectSporeProcess` directly instead of `Execute()`), so the file is never
written. That is the whole reason the mod "does not start" for people who follow
the README.

### 3.2 The ModAPI Launcher, reverse-engineered

`Spore ModAPI Launcher.exe` is a 32-bit .NET assembly. It was read with
reflection and a hand-written IL disassembler (both under
`%WINDIR%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` — a 64-bit host
cannot load it: *"The given program is not a valid Win32 application"*).

Facts extracted from `ModAPI.Common` and from the launcher's IL:

| Item | Value |
|---|---|
| `GameVersion.ExecutableSizes` | 24 909 584→Disc, 24 898 224→Origin_March2017, 24 906 040→EA_October2024, 24 885 248→Steam_March2017, 24 895 536→GOG_October2024, **25 066 744→Steam_October2024** |
| `GetNewDLLName(Steam_October2024)` | `SporeModAPI.march2017.dll` (from `coreLibs\`) |
| `GetVersionName(Steam_October2024)` | `steam_patched` (→ `SporeModAPI-steam_patched.dll`) |
| `GetExecutableFileName(Steam_October2024)` | `SporeApp.exe` (no `SporeApp_ModAPIFix.exe` needed) |
| `RequiresModAPIFix(Steam_October2024)` | **False** — no admin fix required for this build |
| `SporePath.GalacticAdventuresSteamID` | `24720` |
| `SporePath.GetSporebinEP1Path()` | the Steam install (`<Steam root>\steamapps\common\spore\SporebinEP1`) |

The launch flow (`Program.Execute` → `Program.InjectSporeProcess` → `Injector`):

1. `CreateSporeProcess()` — `CreateProcessW` with `CREATE_SUSPENDED`,
   `lpApplicationName = NULL`, `lpCommandLine = "<exe>" <own command line>`,
   `lpCurrentDirectory = SporebinPath`.
2. `Injector.InjectDLL(pi, <kit>\ModAPI.DLLInjector.dll)` — `OpenProcess` →
   `VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread` running
   `LoadLibraryA`.
3. `GetDLLsToInject("steam_patched")` — copies
   `coreLibs\SporeModAPI.march2017.dll` → `mLibs\SporeModAPI.dll` and
   `coreLibs\SporeModAPI.lib` → `mLibs\SporeModAPI.lib`, and returns
   `mLibs\SporeModAPI.dll`, `<kit>\SporeModAPI-steam_patched.dll` and every other
   file in `mLibs` (that is where the mod DLL goes).
4. `Injector.SetInjectionData(pi, hInjector, isDisc=false, dlls)` — the injected
   injector loads the listed DLLs into the suspended game and lets it run.

**Consequence, and the reason for the second kit copy:** step 3 **overwrites**
`mLibs\SporeModAPI.dll` on every launch. A second launch from the same kit copy
fails with *"не удалось скопировать файл … SporeModAPI.dll, так как он
используется другим процессом"* (the first game process holds the file open).
Two concurrent windows therefore need **two separate kit folders** — one per
window. This confirms the upstream scripts' "two Launcher Kit installs" rule, and
explains it.

### 3.3 The Steam "Select Launch Option" dialog

Steam shows a CEF dialog for app 24720 ("Select Launch Option (%1s)",
`LaunchOptionsDialog_SelectHeader` in SteamUI's localization). It is a leftover
from the app's launch options and it clears itself; it is not a blocker and it
was not part of the injection problem.

---

## Phase 4 — Choose

**Path 1: revive/finish `spore-multiplayer-mod`.** Not Path 2 — there is a real,
substantial, MIT-licensed implementation to build on, and rebuilding co-op for
Spore from zero would be a far bigger job.

**Second, smaller decision: what is actually deliverable.**

| Goal | Verdict |
|---|---|
| Two players, one PC, Cell stage | **Achievable and achieved** (Phase 6) |
| Two players, different PCs, Cell stage | Plausible: the transport is already plain TCP and the server is a standalone process. **Not tested** — world transfer between machines is still missing. |
| **Space stage co-op** | **Not achievable with any existing project.** No project implements it, CoopMeld's service is paused and it never got past "Coming later" for Space. This would be new development on top of the Cell-stage base. |

---

## Phase 5 — Build

**No native code was changed, and none could be:** the native DLL (`SporeCoop.Probe.dll`)
is pinned to hashed RVAs and must be recompiled with MSVC v143 to be modified,
and the owner declined the Visual Studio Build Tools. The prebuilt release DLL
was used as-is.

What *was* built here is the missing launcher glue that the upstream scripts do
not contain, plus the knowledge to write it:

- A working launch recipe (see "How to play" in `README.md`) that
  (a) writes `steam_appid.txt` = `24720` into the game's `SporebinEP1`,
  (b) starts the session server with `--listen/--port/--host-token/--guest-token/--save`,
  (c) starts the host window from kit copy 1 and the guest window from kit copy 2,
  (d) points both at the same server and gives them distinct roles/tokens.
- The mod DLL (`bin\SporeCoop.Probe.dll`, SHA-256
  `859FCC53796F8AFCCB2343AC7064144AC23229EF6647D6753A55053C56313C2E`) copied into
  `mLibs\` of **both** kit copies.

The upstream `Start-SporeCoopInstance.ps1` can be reused as-is once
`steam_appid.txt` exists and a second kit copy is present; it was not modified
(it lives in the upstream release, not in this repo).

---

## Phase 6 — QA

### 6.1 The mod's own test suites — actually run

| Suite | Command | Result |
|---|---|---|
| Server protocol | `Test-Server.ps1` (rebuilds `SporeCoop.Server.exe` with the .NET Framework `csc.exe`, then runs it) | **PASS: 142 protocol assertions. No gameplay was tested.** |
| Native source invariants | `node tests/native-source.test.mjs` | **PASS: native lifecycle source invariants.** |
| Native engine checks | `node tests/native-idle.test.mjs` | **cannot run here** — it needs `bin\tests\NativeVisualTests.exe`, a prebuilt binary that is **not** in the release zip and would have to be compiled with MSVC. Blocked by the declined Build Tools. |

### 6.1a This repo's own tooling — also run

| Check | Result |
|---|---|
| `python tools/build_directory.py --check` (what CI runs) | **OK: 302 entries** |
| `python -m unittest tools/mcp/test_coop_mcp.py` (what CI runs) | **Ran 13 tests … OK** |
| `python -I tools/test_all.py chaos fuzz gameslot gamepool scenarios bots animcheck` | `gameslot`, `gamepool`, `scenarios`, `bots`, `animcheck` **pass**. `chaos` and `fuzz` report FAIL **only because cmake is not installed here**: `fuzz` cannot build (`FileNotFoundError: cmake`), and `chaos` exits 0 but its last line is `ALL OK (C++ cross-check skipped: cmake not found on PATH))`, which the runner does not accept as exactly `ALL OK`. `flight`, `ownership` and `fakeworld` were not run for the same reason. |

These suites test **this repo's own sync-testing tools**, which this game does not
use — the Spore mod has its own protocol and its own test scripts. The two
commands CI actually runs are both green, so the PR is not blocked by this.

### 6.2 End-to-end injection test (this session) — the real result

Run on the **Steam October 2024** build, from the **test copy**, with the mod's
**prebuilt** release DLL and no source changes.

**Setup**

```
<game>\SporebinEP1\steam_appid.txt          = 24720          (test copy only)
SporeCoop.Server.exe --listen 127.0.0.1 --port 5523 \
        --host-token <host> --guest-token <guest> --save <path>\session.json
Start-SporeCoopInstance.ps1 -Profile 1 -Role host  -LauncherRoot "<kit 1>" -GameRoot "<test copy>" ...
Start-SporeCoopInstance.ps1 -Profile 2 -Role guest -LauncherRoot "<kit 2>" -GameRoot "<test copy>" ...
```

**Observed results**

1. Both game processes stayed alive, launched from the **test copy**
   (`<test copy>\SporebinEP1\SporeApp.exe`), not from the Steam install.
2. The mod's probe log (`%TEMP%\SporeCoop.Probe.log`) shows, for **both** windows:

   ```
   [pid=8252  role=host ] Native build: Sep 27 2026 17:43:00; registered peer renderer.
   [pid=8252  role=host ] Initialize completed; commands registered and cooperative client started.
   [pid=8252  role=host ] Cooperative connection state reset after handshake.
   [pid=20468 role=guest] Profile 2 paths selected: SporeCoop2 and My Spore Creations Coop 2.
   [pid=20468 role=guest] Initialize completed; commands registered and cooperative client started.
   [pid=20468 role=guest] Cooperative connection state reset after handshake.
   ```

3. The session server's own stdout (`local/testrig/server.log`):

   ```
   SporeCoop Version 1 server (protocol 10). Supported stage: Cell.
   HOST TOKEN (keep local): …
   GUEST TOKEN (share with friend): …
   LISTENING 127.0.0.1:5523
   Connected: host
   Connected: guest
   ```

4. Two **Established** TCP sessions on port 5523, both owned by the server
   process, plus the listening socket.
5. The server wrote `session.json` (`"Stage":"cell"`, `"Format":1`,
   `"Revision":0`, `"Applied":[]`).

   Captured in `local/testrig/` (`probe-2026-10-21.log`, `server.log`,
   `session.json`).

**Conclusion of 6.2:** the mod's native client *does* load into this game build
and *does* complete a protocol handshake with the session server from two
separate game processes. That had never been demonstrated before.

### 6.3 What is still **not** verified

- **Two players in the same world.** The invite flow is in-game UI: the host
  clicks "Invite", picks a player in a picker window, the guest clicks "Accept"
  in a prompt (`native/Probe.cpp`, `kInviteButtonID` / `kInvitePlayerButtonID` /
  `kAcceptInviteButtonID`). Driving that needs either a human or a UI bot; the
  diagnostic commands (`coopStatus`, `coopSpawn`, `coopJoin`) are registered
  through `CheatManager.AddCheat`, i.e. they are only reachable from the game's
  cheat UI, and there is no scriptable trigger.
- The end screen, saving, return to the menu, and fragment drops — the author's
  own list of unconfirmed behaviour.
- Remote play between two machines (world transfer is not implemented).
- The 154 native engine checks (`NativeVisualTests.exe` not buildable here).

### 6.4 The Space-stage gap, in the mod's own words

The probe log of the running game says, for both windows:

```
Version 1: verified Cell-stage ending after History; further stages disabled in co-op.
```

and the session server prints `Supported stage: Cell.`

That is in-game, server-side proof that **co-op ends after the Cell stage** —
the four later stages (Creature, Tribal, Civilization, Space) have no co-op at
all in this project. The owner's stated priority (Space stage) is therefore
**not covered by any project that exists**.

### 6.5 A note on module enumeration

`Get-Process … .Modules` from a 64-bit PowerShell into the 32-bit game process
lists only **7** modules — the WoW64 host layer (`wow64.dll`, `wow64base.dll`,
…) — and never shows the ModAPI DLLs. The module list is **not** usable as
injection evidence here; the probe log is, because it is written by the mod's own
code inside the game process and carries the game PID.

---

## What to tell the owner

1. **Good news:** the emerald mod *does* work on this exact Steam build once it
   is launched correctly. Two windows, one PC, both connected to the session
   server — verified. The missing piece was never a broken mod; it was a
   missing `steam_appid.txt` and two kit copies, neither of which the upstream
   README explains.
2. **Bad news:** that is the **Cell** stage only, on **one PC**. The Space
   stage has no co-op implementation anywhere, and the one project that aimed
   higher (CoopMeld) has a paused service (HTTP 503).
3. **Realistic options:**
   a. Ship the Cell-stage mod as a working local/remote co-op (needs: a world
      transfer step, a reachable server, and one interactive in-game session to
      confirm the invite flow).
   b. Port/implement the later stages on top of it — real engineering, and the
      Space stage is a different game (a 3D galaxy flight with planets and
      cities) from the 2D cell stage the mod handles.
   c. Wait for CoopMeld to come back — but it had not reached Space either.
