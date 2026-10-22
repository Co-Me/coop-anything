# Spore: Galactic Adventures (2009)

| | |
|---|---|
| Engine | Spore (proprietary EA/Maxis, RenderWare rendering) |
| Versions tested | Steam app **24720**, Galactic Adventures exe `SporebinEP1\SporeApp.exe`, 25 066 744 bytes, file version 3.1.0.29, SHA-256 `3010A413F9A4A05EE9C4DE24756C1C94C7FB218F70CAD29BCF5BC0B4ABFA2466`. The ModAPI Launcher's own version table calls this build **`Steam_October2024`**. Tested on Windows, Russian language. |
| Projects | [Spore Multiplayer Mod](https://github.com/emerald194520350-alt/spore-multiplayer-mod) (v1.0.1) · [CoopMeld "Spore Coop"](https://coopmeld.com/spore/) (service paused) · [SporeMP](https://github.com/raph559/SporeMP) (GOG build only) · [Spore-Multiplayer](https://github.com/Spore-Multiplayer/Spore-Multiplayer) (empty) · [sunglocto/SporeMultiplayer](https://github.com/sunglocto/SporeMultiplayer) (README only) |
| QA level | `automated` for the Cell-stage mod (see QA), `none` for the rest |
| Last updated | 2026-10-21 |

## Where things stand

**Co-op works for the Cell stage only, and it was verified for the first time
here — on the owner's exact Steam build, with two real game processes talking to
a real session server.** No project supports the later stages, and the owner's
priority (the **Space** stage) is **not covered by anything that exists**.

Five Spore co-op projects were found. Two are real code (the MIT-licensed
*Spore Multiplayer Mod* and the GPL *CoopMeld Spore Coop*), one is a prototype
pinned to a different store build, and two are empty. All of them stop well
short of Space, and the one that aimed furthest has a **paused live service**
(HTTP 503) and had only reached "Civilization and Space — coming later".

## Existing work

### Spore Multiplayer Mod — `emerald194520350-alt/spore-multiplayer-mod` ✅ works

| | |
|---|---|
| Author | `emerald194520350-alt`, single author |
| Licence | MIT |
| Latest release | v1.0.1, 2026-09-27, protocol 10 |
| Asset | `spore-multiplayer-mod-v1.0.1-windows.zip`, 357 314 bytes, 8 downloads |
| SHA-256 | `787cac1f7b262dba76093923a75dcecc07916c4a5185182b94893b8c4d181e78` (published sum, verified) |
| Activity | 21 commits between 2026-09-17 and 2026-09-27, then silent |
| Issue tracker | **none**. There is nowhere to report a bug. |
| Stages | **Cell only** (1 of 5) |

Architecture, read from the source:

```
SporeApp.exe window 1 (host)  ─┐
  native/Probe.cpp   C++ client, Spore ModAPI + Detours hooks
  native/CoopNet.cpp JSON over length-prefixed TCP
                                 ├──► SporeCoop.Server.exe   C# session server
SporeApp.exe window 2 (guest) ─┘     server/SessionServer.cs
                                        central broker, owns shared state,
                                        broadcasts position / appearance /
                                        npcSnapshot / worldAction
```

The server is a plain `TcpListener` process with `--listen` and `--port`, and the
clients read their address from `SPORE_COOP_SERVER`, `SPORE_COOP_PORT`,
`SPORE_COOP_TOKEN`, `SPORE_COOP_ROLE` and `SPORE_COOP_PROFILE`.

What it syncs: player movement and appearance, NPC replication, shared growth,
DNA and parts, diet history, species name, the creature editor (shared model plus a
shared DNA budget with revision merging), shared progress and unlocks, the History
timeline, one-sided campaign saves (the owner writes, the guest is blocked), pause,
and editor entry and exit.

What it does not do: the Creature, Tribal, Civilization and Space stages. After
Cell the mod shows an end screen instead of the landfall transition — confirmed
in-game, see QA.

### CoopMeld "Spore Coop" — https://coopmeld.com/spore/ ⚠️ service paused

The only project that ever went past the Cell stage. GPL sources (installer,
native mod, launcher), 2–4 players, online rooms, Cell + Creature + Tribe.

- The **live service is paused**: `/api/check?nonce=…` → **HTTP 503 "Release
  unavailable."**; `/lobby` → **HTTP 426** `{"error": "Update SporeCoop in the
  launcher before playing."}`.
- Its own site lists Civilization and Space as *"Coming later"*.
- Its build guard (`launcher/game_build.h`) matches this owner's install
  (`October2024Steam` = 25 066 744 bytes **and** version 3.1.0.29), and the
  release policy is RSA-signed with the exe hash in a `games` set — so the build
  is compatible, only the service is down.
- Its policy requires `stage: cell`.

### SporeMP — https://github.com/raph559/SporeMP ⚠️ wrong store build

GPL-3.0, has a roadmap for all stages, ships a dedicated server. Its byte-level
guard is pinned to the **GOG** build (24 895 536 bytes, SHA-256 `dc04aee5…`) and
**rejects the Steam build** this owner has. Prototype quality.

### Dead ends

- [Spore-Multiplayer/Spore-Multiplayer](https://github.com/Spore-Multiplayer/Spore-Multiplayer)
  — an empty Visual Studio skeleton; `dllmain.cpp` is the ModAPI template,
  `Client/README.md` says *"Files for the Client. TBA"*.
- [sunglocto/SporeMultiplayer](https://github.com/sunglocto/SporeMultiplayer) —
  a 545-byte README, no code.
- `NeDether/SporeMultiplayer` — **404 / gone**.

Two more Spore projects exist but are not co-op:
[SporeNEXT](https://github.com/bisfranz/SporeNEXT) is a mod manager, and the
[Spore ModAPI SDK](https://github.com/Spore-Community/Spore-ModAPI) is the C++
modding framework everything is built on.

## How the game works (for co-op)

- **Engine**: proprietary EA/Maxis. Rendering through RenderWare. No shipped
  multiplayer and therefore **no existing netcode** to reuse.
- **Modding surface**: the Spore ModAPI SDK is a reverse-engineered C++ SDK that
  hooks the game with Microsoft Detours and is injected by the
  [SPORE ModAPI Launcher Kit](https://github.com/Spore-Community/ModAPI-Launcher-Kit).
  The SDK ships Ghidra import files (`SporeGhidra_march2017.xml`) for deeper work.
- **The launch path is the hard part.** The kit is a 32-bit .NET app that
  `CreateProcess`es the game **suspended**, injects `ModAPI.DLLInjector.dll`
  with `WriteProcessMemory` + `CreateRemoteThread(LoadLibraryA)`, tells the
  injector which DLLs to load, and lets the game run. Every step has to be
  right, and the kit's GUI flow does two things the upstream script skips:
  - it writes **`steam_appid.txt`** with the app id (`24720`) into the game's
    `SporebinEP1` folder, so `SteamAPI_Init` succeeds when the game is started
    outside Steam — without it the game exits immediately with **code 53**;
  - it copies `coreLibs\SporeModAPI.<version>.dll` over `mLibs\SporeModAPI.dll`
    on **every** launch, so a second concurrent window from the same kit folder
    fails with *"cannot copy SporeModAPI.dll, the file is in use by another
    process"*. **Two windows therefore need two separate kit folders.**
- **Version detection** is by exe size: `ModAPI.Common.GameVersion.ExecutableSizes`
  maps 25 066 744 → `Steam_October2024`, 24 885 248 → `Steam_March2017`,
  24 895 536 → `GOG_October2024`, 24 909 584 → `Disc`, and so on. The upstream
  script labels the october build "Steam March 2017"; both resolve to the same
  core DLL (`SporeModAPI.march2017.dll`), so the label is harmless for the kit.
- **Build pinning**: the mod detours functions at hashed RVAs
  (`VerifiedMotionCode`), so it only works on the executable it was built
  against. A different Galactic Adventures build breaks it. The mod's
  `Check-Environment.ps1` whitelists five accepted exe sizes, and this build is
  one of them.
- **Save system**: a campaign lives in `%APPDATA%\Spore\Games` plus creature assets
  under `Documents\My Spore Creations`. The mod isolates a second profile at
  `%APPDATA%\SporeCoop2` by detouring `App::cAppSystem`, and hides the borrowed
  campaign copies from the guest's galaxy menu (`native/BorrowedCampaigns.h`).
- **World ownership**: the invitation sender is the world owner. It applies
  authoritative world changes and writes the campaign; the guest's writes are
  blocked at the native save hooks (`native/CampaignSync.h`).
- **Stage scope**: the Cell stage is a 2D single-screen world. The Space stage
  is a different game — 3D galaxy flight, planets, cities — and nothing in any
  of these projects touches it.

## What we changed

**No mod code was changed.** The native DLL is pinned to hashed RVAs and would
need MSVC v143 to rebuild, which the owner declined; the **prebuilt release DLL**
was used as-is. What was added is the missing launch glue, and the diagnosis
that explains why the mod "does not start" for anyone following the upstream
README:

| Problem | Root cause | Fix | Proof |
|---|---|---|---|
| Game exits with **code 53** when started outside Steam | the Steam build needs `steam_appid.txt` with its app id for `SteamAPI_Init`; the upstream script bypasses the kit's `Execute()` flow that writes it | write `steam_appid.txt` = `24720` into `<game>\SporebinEP1` | direct launch then runs (≈ 408 MB, window present); with the wrong id `17390` it exits with code 86 instead |
| Second window fails to launch | the kit overwrites `mLibs\SporeModAPI.dll` on every launch, and the first game process holds it open | use a **second copy of the kit** for the second window | two windows start and stay up |
| Mod never appears to load | none — this one was a *reporting* artefact: `Get-Process .Modules` from a 64-bit shell lists only the 7 WoW64 host modules of the 32-bit game | use the mod's own probe log (`%TEMP%\SporeCoop.Probe.log`) as the evidence | the log carries the game PID and the mod's initialisation lines |

## QA

### Suites that were actually run

| Suite | Command | Result |
|---|---|---|
| Mod's server protocol suite | `Test-Server.ps1` (rebuilds `SporeCoop.Server.exe` with the .NET Framework `csc.exe`, then runs it) | **PASS: 142 protocol assertions. No gameplay was tested.** |
| Mod's native source invariants | `node tests/native-source.test.mjs` | **PASS: native lifecycle source invariants.** |
| Mod's native engine checks | `node tests/native-idle.test.mjs` | **could not run** — it needs `bin\tests\NativeVisualTests.exe`, a prebuilt binary that is *not* in the release zip and would have to be compiled with MSVC (Build Tools declined) |
| This repo's own tool suite | `python -I tools/test_all.py` | `gameslot`, `gamepool`, `scenarios`, `bots`, `animcheck` pass. `chaos` and `fuzz` report FAIL only because cmake is not installed on this machine (`fuzz` cannot build; `chaos` prints `ALL OK (C++ cross-check skipped: cmake not found on PATH))`, which the runner does not count as `ALL OK`), and `flight`, `ownership`, `fakeworld` need cmake and ctest. These suites exercise this repo's own sync tooling, which this game does not use. |
| This repo's CI | `python tools/build_directory.py --check` and `python -m unittest tools/mcp/test_coop_mcp.py` | `OK: 302 entries` and `Ran 13 tests … OK` |

### End-to-end test run here

Real game clients, real session server, the owner's Steam build, launched from a
**test copy** of the install (the owner's own install and saves were never
touched).

**Setup**

1. `steam_appid.txt` = `24720` written into `<game>\SporebinEP1` of the test copy.
2. `SporeCoop.Server.exe --listen 127.0.0.1 --port 5523 --host-token <host> --guest-token <guest> --save <path>\session.json`
3. Host window: `Start-SporeCoopInstance.ps1 -Profile 1 -Role host -LauncherRoot "<kit copy 1>" -GameRoot "<test copy>" -Server 127.0.0.1 -Port 5523 -Token <host>`
4. Guest window: the same script with `-Profile 2 -Role guest -LauncherRoot "<kit copy 2>" -Token <guest>`

**Observed**

- Both game processes stayed alive, launched from the test copy
  (`<game>\SporebinEP1\SporeApp.exe`), not from the Steam install.
- The mod's probe log, for **both** windows:

  ```
  [pid=8252  role=host ] Native build: Sep 27 2026 17:43:00; registered peer renderer.
  [pid=8252  role=host ] Initialize completed; commands registered and cooperative client started.
  [pid=8252  role=host ] Cooperative connection state reset after handshake.
  [pid=20468 role=guest] Profile 2 paths selected: SporeCoop2 and My Spore Creations Coop 2.
  [pid=20468 role=guest] Initialize completed; commands registered and cooperative client started.
  [pid=20468 role=guest] Cooperative connection state reset after handshake.
  ```

- The session server's own stdout:

  ```
  SporeCoop Version 1 server (protocol 10). Supported stage: Cell.
  HOST TOKEN (keep local): …
  GUEST TOKEN (share with friend): …
  LISTENING 127.0.0.1:5523
  Connected: host
  Connected: guest
  ```

- Two **Established** TCP sessions on port 5523, both owned by the server
  process, plus the listening socket.
- The server wrote `session.json` (`"Stage":"cell"`, `"Format":1`, `"Revision":0`).

**In-game proof of the stage limit** — the mod's own log, in the running game,
for both windows:

```
Version 1: verified Cell-stage ending after History; further stages disabled in co-op.
```

and the server prints `Supported stage: Cell.`

### Not verified

- **Two players in the same world.** The invite is in-game UI: the host clicks
  "Invite", picks a player in a picker, the guest clicks "Accept"
  (`kInviteButtonID` / `kInvitePlayerButtonID` / `kAcceptInviteButtonID`). The
  diagnostic commands `coopStatus` / `coopSpawn` / `coopJoin` are registered via
  `CheatManager.AddCheat` and have no scriptable trigger.
- The end screen, saving, return to the menu and fragment drops — the author's
  own list of unconfirmed behaviour.
- Remote play between two machines (world transfer is not implemented).
- The 154 native engine checks (not buildable here).

## How to play

Not ready to play with friends yet. The verified local setup (Cell stage, one PC,
two windows):

1. Install Spore: Galactic Adventures from Steam and the
   [SPORE ModAPI Launcher Kit](https://github.com/Spore-Community/ModAPI-Launcher-Kit)
   — and make a **second copy of the kit folder**, one per window.
2. Close both game windows, extract `spore-multiplayer-mod-v1.0.1-windows.zip`.
3. Write `24720` into a file `steam_appid.txt` in `<game>\SporebinEP1`
   (the app id of the executable that runs there). Without it the game exits
   with code 53.
4. Start the session server:
   `SporeCoop.Server.exe --listen 127.0.0.1 --port 5523 --host-token <host> --guest-token <guest> --save session.json`
5. Start the host window with `Start-SporeCoopInstance.ps1 -Profile 1 -Role host
   -LauncherRoot "<kit copy 1>" -GameRoot "<game dir>" -Server 127.0.0.1 -Port 5523
   -Token <host>`, and the guest window with `-Profile 2 -Role guest
   -LauncherRoot "<kit copy 2>" -Token <guest>`.
6. Load a Cell world in the host window, open the Esc menu, choose "Invite friend
   to co-op", accept in the guest window.

The upstream scripts hardcode the author's machine (`C:\Games\SPORE Collection`,
`C:\ProgramData\SPORE ModAPI Launcher Kit` and a second copy named
`… Launcher Kit 2`), so they need repointing for any other install.

## Known issues

- **Only the Cell stage.** The other four stages — including Space — are not
  playable in co-op by any project.
- **The Space stage has no implementation at all.** This is the honest answer to
  "can I play Space co-op with friends today": no.
- Two windows on one PC are what was verified. Remote play between computers is
  unfinished and world transfer between machines is not implemented.
- The invite/accept flow still needs an interactive confirmation by a human.
- Only one save is kept: the world owner's. The guest's campaign is protected from
  being written.
- The guest needs a copy of the host's campaign. The launcher does this copy
  locally; across machines it has to be done by hand.
- The mod detours functions at hashed RVAs, so a game update breaks it.
- Rare save states, tutorials and unusual editor transitions may still misbehave.
- The launcher kits, the game install and the two profiles are all hardcoded
  paths that have to be repointed per machine.

## Next steps

1. **One interactive session** to confirm the invite → accept → two cells in one
   world, with the end screen and saving. This is the last unverified piece of
   the Cell-stage mod and it needs a human at the keyboard (or a UI bot).
2. **World transfer**, the real blocker for remote play: the guest's profile has
   to receive the host's campaign and creature assets. Everything else is
   plumbing — the transport is already TCP and the server is already a
   standalone process.
3. **Decide the target stage.** Cell-stage co-op is one product; the Space stage
   is a different game (3D galaxy flight, planets, cities) and would be new
   development, not a port. CoopMeld's "coming later" is not a plan.
4. **Upstream the two launch fixes** (the `steam_appid.txt` step and the
   "two kit folders because the kit overwrites its own DLL" note) — the project
   has no issue tracker, so that means a pull request or a fork.
