# X-Wing Cockpit

**A touch cockpit control panel for the classic LucasArts space sims, built for the
AYN Thor dual-screen handheld.**

https://youtu.be/om3cUkTnZiw

Version 1.0

X-Wing Cockpit runs on the Thor's **bottom** screen as a themed control panel while
*X-Wing*, *TIE Fighter*, *X-Wing vs TIE Fighter*, or *X-Wing Alliance* runs in
GameNative/Winlator on the **top** screen. Tapping a control on the panel presses
the matching keyboard key in the game, so systems management, shield juggling,
wingman orders, and mission goals are all one thumb-tap away instead of buried
under keyboard shortcuts you'd never reach on a handheld.

---

## Contents
- [How it works](#how-it-works)
- [Requirements](#requirements)
- [First-run setup](#first-run-setup)
- [Using the cockpit](#using-the-cockpit)
- [Games, craft & missions](#games-craft--missions)
- [Controls reference](#controls-reference)
- [Building from source](#building-from-source)
- [Project structure](#project-structure)
- [Extending the app](#extending-the-app)
- [Known limitations](#known-limitations)
- [1.0 highlights](#10-highlights)

---

## How it works

Android sandboxes apps, so one app cannot type into another. X-Wing Cockpit gets
around this with **Shizuku**, which runs a small helper service in a
shell-privileged process. That process holds Android's `INJECT_EVENTS` permission,
so it can inject key events that reach whatever window currently has input focus —
the game.

Two design choices make this reliable on a dual-screen device:

1. **The panel never takes input focus.** The app's window is marked
   `FLAG_NOT_FOCUSABLE`. It still receives your touches (all the buttons work), but
   it never steals focus from the game. That keeps the game's own controller input
   alive *and* ensures injected keys land in the game rather than the panel.

2. **Keys are injected like a real keyboard.** The helper calls
   `InputManager.injectInputEvent` and reproduces a genuine physical-keyboard
   sequence — for a modified command it sends *Shift down → letter down → letter up
   → Shift up*, holding each key ~40 ms so the game reliably polls it. This is what
   makes Shift-based commands (wingman orders, docking) work through the Wine layer,
   where a naive "key combination" call would drop the modifier.

Touch controls fire on **press-down** (not on tap-release), so a quick or light tap
can never flash without sending.

---

## Requirements

- **AYN Thor** (or any Android 8.0+ / API 26+ device; dual-screen assumed).
- **Shizuku** installed and running. No root required — Shizuku can be started over
  wireless debugging / ADB. (Rooted devices can start it with one tap.)
- **GameNative** (or Gamehub or Winlator build) running the game on the primary screen,
  with the game's controls set to **keyboard** and keyboard input passed through to
  the game.
-  A legally obtained copy of a game in the X-wing series.  Steam, GOG, an .ISO from your own disc with Winlator.

---

## First-run setup

1. **Start Shizuku** on the Thor (wireless-debugging method if unrooted; re-run
   after each reboot unless rooted).
2. **Launch X-Wing Cockpit** on the bottom screen. Open **CONFIG → Diagnostics**,
   tap **Grant / Connect**, and approve Shizuku's prompt. Status should read
   **READY — key link is live**.
3. **Launch your game** in GameNative on the top screen and get into flight.
4. **Verify the link:** with the game focused, tap the Diagnostics **S (shields)**
   test button. If the shields reconfigure in-game, the pipeline works and every
   other control will too.

Because the panel is non-focusable, injected keys always go to whatever is focused
on the main screen. To confirm injection you watch the *game* react — there is no
in-app capture box (a non-focusable window can't host one).

---

## Using the cockpit

**Navigation:** Game → Faction → Ship → *Mission* → Loadout → Cockpit.
The Faction step is skipped for single-side games (X-Wing = Rebel, TIE Fighter =
Imperial). The Mission step appears for the three games that ship mission data
(X-Wing, TIE Fighter, X-Wing Alliance); pick **No Battle / Custom Mission** to skip
goal tracking.

**The cockpit** is three columns:

- **Left** — vertical recharge meters (Laser, Shield, Engine, Beam) with the
  **COMMANDS** and **CONFIG** buttons tucked underneath. Laser fill is **red for
  Rebel craft, green for Imperial**, matching each side's laser color.
- **Middle** (top-down stack) — the ship silhouette with a tappable shield-arc
  indicator, the color-coded **weapon** button (green = lasers, blue = ions, red =
  warheads), the **fire-mode** cannon-dot graphic, and the two **energy-transfer**
  buttons.
- **Right** — the throttle column (Full / Match / 2-3 / 1-3 / Zero) plus the SLAM
  button on craft that have it.

**COMMANDS** opens a full-screen overlay of ship, wingman, docking, and display
commands. **CONFIG** holds Switch Craft, Reset Cockpit, and Diagnostics.

**Mission goals:** COMMANDS → DISPLAYS → **Mission Goals** sends `g` (opening the
game's own goals menu) and, when a mission is selected, opens the app's goals
screen showing that mission's objectives, **hidden bonus goals**, and tips. Its
CLOSE button sends `g` again to dismiss the in-game menu and returns you to the
commands view.

Because the game never reports its state back, the panel tracks an **assumed**
model seeded from each craft's known starting configuration. If you also press keys
on a controller/keyboard the assumption can drift; **Reset Cockpit** re-syncs it.

---

## Games, craft & missions

**Games:** X-Wing (Rebel), TIE Fighter (Imperial), X-Wing vs TIE Fighter (both
sides), X-Wing Alliance (both sides). Each game has its own keymap; the only
divergence today is XWA's zero-throttle key (`/` vs `\`).

**Craft (12):**

| Rebel | Imperial |
|-------|----------|
| X-Wing | TIE Fighter |
| Y-Wing | TIE Interceptor |
| A-Wing | TIE Bomber |
| B-Wing | TIE Advanced |
| Z-95 Headhunter | Assault Gunboat |
| | TIE Defender |
| | Missile Boat |

Each craft appears only in the games it belongs to, with the correct cannon count,
shield/ion/beam/SLAM capabilities, and a bundled silhouette. The panel shows or
hides controls to match — e.g. shieldless TIEs have no Shield meter or transfer
buttons; the Missile Boat carries two warhead types.

**Missions & bonus goals** (bundled as `assets/missions.json`):

| Game | Groups | Missions | Notes |
|------|--------|----------|-------|
| X-Wing | 6 tours | 100 | Training Ground (by craft) + 5 Tours of Duty; summary → goals → tips |
| TIE Fighter | 13 battles | 76 | Hidden bonus goals with an Easy / Medium / Hard toggle |
| X-Wing Alliance | 9 battles | 53 | Objectives, hidden bonus goals, opponents, summarized difficulty & tips |

---

## Controls reference

All four games share one keymap unless noted.

### Cockpit
| Control | Key |
|---------|-----|
| Cycle shields (EQUAL → REAR → FORWARD) | S |
| Cycle weapon | W |
| Cycle fire mode | X |
| Laser recharge | F9 |
| Shield recharge | F10 |
| Beam recharge | F8 |
| Transfer laser → shields | `'` |
| Transfer shields → lasers | `;` |
| SLAM | N |

### Throttle
| Button | Key |
|--------|-----|
| Full | Backspace |
| 2/3 | ] |
| 1/3 | [ |
| Zero | `\` (XWA: `/`) |
| Match speed | Enter |

### Commands overlay
| Group | Commands (key) |
|-------|----------------|
| Ship | Next Target (T), Prev Target (Y), Nearest Fighter (R), Enter Hangar (Space), Engage Hyperdrive (H), Cockpit Display (I) |
| Wingman Orders | Attack (Shift+A), Ignore (Shift+I), Evade (Shift+E), Wait/Hold (Shift+W), Go/Proceed (Shift+G), Hyper Home (Shift+H), Report In (Shift+R) |
| Docking / Cargo | Dock (Shift+D), Rearm Me (Shift+B), Pick Up Obj (Shift+P) |
| Displays | Mission Goals (G), Message Log (L), Damage (D), In-Flight Map (M), Target Threat (Z), Keyboard Ref (K) |

Energy transfer uses the single-key `'` / `;` equivalents rather than the
Shift+F10 / Shift+F9 chords, because the single keys are reliable through Wine.

---

## Building from source

Open the project in **Android Studio** and Run onto the device, or use the Gradle
wrapper once it's present:

```
./gradlew assembleDebug
```

**Toolchain (pinned and matched):**

| Component | Version |
|-----------|---------|
| Android Gradle Plugin | 8.13.2 |
| Gradle | 8.13 |
| Kotlin | 1.9.24 |
| Compose compiler extension | 1.5.14 |
| compileSdk / targetSdk | 34 |
| minSdk | 26 |
| JDK | 17 |

AGP and Gradle are version-locked together — if you bump one, bump the other
(AGP 8.13 requires Gradle ≥ 8.13). Kotlin 1.9.24 and Compose compiler 1.5.14 are a
matched pair; keep them in step if you upgrade.

---

## Project structure

```
app/src/main/
├── java/com/example/xwingcockpit/
│   ├── MainActivity.kt          Navigation + the FLAG_NOT_FOCUSABLE window setup
│   ├── model/
│   │   ├── Domain.kt            Games, craft roster, keymaps, energy model,
│   │   │                        fire-mode logic, command groups
│   │   └── Missions.kt          Mission data model + assets/missions.json loader
│   ├── ui/
│   │   ├── CockpitViewModel.kt  Assumed-state model + key dispatch
│   │   └── Screens.kt           All Compose UI
│   └── input/
│       ├── ShizukuInjector.kt   Binds the Shizuku user service, queues key steps
│       ├── UserService.kt       Runs shell-side; injects real keyboard sequences
│       └── aidl/…/IUserService.aidl
├── assets/missions.json         X-Wing (100) + TIE (76) + XWA (53) mission data
└── res/drawable-nodpi/          12 bundled cockpit_*.png craft silhouettes
```

---

## Extending the app

**Add or change craft / games:** edit `model/Domain.kt` only. Add a `Ship(...)` to
`SHIPS` with its `games` set, faction, cannon count, and capability flags; the UI
and injector adapt automatically. Each `Game` references a keymap — diverge one
game with e.g. `CLASSIC_KEYMAP.copy(fireMode = KeyStep(...))`.

**Add craft art:** drop a 1024×1024 transparent PNG into `res/drawable-nodpi/`
named to match the ship's `art` id (e.g. `cockpit_xwing.png`).

**Edit missions:** regenerate or hand-edit `assets/missions.json`. Each game maps
to a list of battles/tours, each with missions carrying goals, bonus goals, and
(for X-Wing/XWA) summary and tips.

**Rebind commands:** the wingman/ship/docking/display commands live in
`COMMAND_GROUPS` in `model/Domain.kt`.

---

## Known limitations

- **No live game state.** The panel can't read the game, so it shows an assumed
  model. Use **Reset Cockpit** to re-sync after using another input device.
- **No hardware-button shortcut to the goals screen.** Because the panel is
  deliberately non-focusable (the thing that keeps the game focused), it can't
  observe controller/keyboard keys, so a physical button can't toggle the app's
  goals view without an accessibility service or input-monitoring setup. Left out
  by design; use the on-screen Mission Goals command instead.
- **Keyboard passthrough depends on GameNative.** Injected keys only help if the
  game's controls are set to keyboard and GameNative forwards keyboard input to it
  (the same path a real Bluetooth keyboard would use).

---

## 1.0 highlights

Everything below is implemented and confirmed on real AYN Thor hardware:

- Full cockpit control panel for all four games, laid out for the Thor's near-square
  bottom screen (top-down middle column, per-faction laser colors, color-coded
  weapon button).
- Reliable key injection via `InputManager.injectInputEvent`, including true
  Shift-chord commands and a ~40 ms key hold so nothing is dropped.
- Non-focusable window so the game keeps focus and controller input.
- Touch controls fire on press-down for responsiveness.
- Single-key energy transfer (`'` / `;`); correct shield-cycle direction.
- Mission-goal system across three games (229 missions total) surfacing hidden
  bonus objectives, with per-difficulty goals for TIE Fighter and summarized tips
  for X-Wing Alliance.
- All 12 craft silhouettes bundled; no per-build asset copying.
