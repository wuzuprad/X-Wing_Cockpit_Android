# X-Wing_Cockpit_Android
An android companion app for the Star Wars X-wing series.  This is specifically a dual screen cockpit companion for dual screen android handhelds
# X-Wing Cockpit — companion control panel (AYN Thor bottom screen)

A themed touch control panel that sends keystrokes to X-Wing Alliance (and the
other classic X-Wing / TIE Fighter games) running in Winlator. Tapping a cockpit
control presses the matching key in the game.

## How it sends keys (important — read first)

The app can't type into the game directly; Android sandboxes apps. It uses
**Shizuku** to run a tiny helper in a shell-privileged process, which calls the
system `input` command. Those key events are injected globally and reach the
focused window — i.e. the game.

Because injected keys go to the **focused** window, the game must keep keyboard
focus while you tap the bottom screen. On a dual-screen device this usually works,
but it is exactly the thing that can vary by device — which is why you validate it
in the **Diagnostics** screen before trusting the cockpit. If keys land in the
wrong place, that's the knob to investigate (making this app's window
non-focusable is the likely fix; ask and I'll add it).

## First-run workflow (do this before anything else)

1. Install **Shizuku** (from its GitHub / Play) on the Thor and start it. No root:
   start it via wireless/ADB per Shizuku's own instructions. With root: one tap.
2. Build & install this app (below). Launch it on the **bottom** screen.
3. Open **Diagnostics**. Tap *Grant / Connect* and approve Shizuku's prompt.
   Status should read **READY**.
4. Launch X-Wing Alliance in Winlator on the **top** screen. Get into flight.
5. With the game focused, tap the **S** test button. If the shields reconfigure,
   the pipeline works and every other control will too. If not, tell me what
   happened and we adjust the injection method.

Only after the S test works is it worth wiring in real cockpit art.

## Build

Open the folder in **Android Studio** (Hedgehog or newer). It will download the
Gradle wrapper and dependencies. Then Run onto the device, or `./gradlew
assembleDebug` once the wrapper is present.

- compileSdk 34, minSdk 26.
- `input keycombination` (used for Shift+F9 etc.) needs Android 12+ at runtime.
  The Thor is new hardware so this is fine.
- This is a first build cut written without an on-device compile pass. Expect to
  fix a stray import or version nudge in Android Studio; send me anything it flags.

## Games & selection flow

Four games are configured: X-Wing, TIE Fighter, X-Wing vs TIE Fighter, and
X-Wing Alliance. Flow is **Game → Faction → Ship**. The faction step is skipped
automatically when a game has one side (X-Wing = Rebel only, TIE Fighter =
Imperial only; XvT and XWA offer both). Ship art is shared per craft type across
all games.

## Verified key bindings (shared classic scheme, all four games)

| Control          | Key          |
|------------------|--------------|
| Cycle shields    | S            |
| Cycle weapons    | W            |
| Cycle fire mode  | X            |
| Laser recharge   | F9 (cycles)  |
| Shield recharge  | F10 (cycles) |
| Beam recharge    | F8 (cycles)  |
| Laser → shields  | Shift+F9 (or ') |
| Shields → lasers | Shift+F10 (or ;)|
| Toggle beam      | B            |
| Full throttle    | Backspace    |
| Match speed      | Enter        |
| 2/3 throttle     | ]            |
| 1/3 throttle     | [            |
| Zero throttle    | \ (XWA: /)   |

Throttle sits in a right-hand column (Full → Match → 2/3 → 1/3 → Zero), defaulting
to Full — the currently selected level glows bright green, the rest are dark red.
Match Speed is treated as its own state (the resulting speed is unknown to the
app). Backspace and Enter are layout-stable; the bracket/slash keys are punctuation
and worth confirming in-game like the transfer keys. Zero throttle is `\` in the
older games and `/` in XWA — handled by XWA's own keymap.

Energy transfer defaults to **Shift+F9 / Shift+F10** — function keys are
layout-independent through Winlator, so they're the more reliable choice. The `'`
and `;` keys do the same thing and are the fallback. The Diagnostics screen fires
all four so you can confirm which actually moves energy in each game, then set the
winner in `KeyMap` (`model/Domain.kt`). Original X-Wing has no beam-capable craft,
so no beam controls appear there.

Confirm in-game and adjust in `model/Domain.kt` if needed:
- `SHIELD_CYCLE_ORDER` — exact order S cycles (default EQUAL→FWD→REAR).
- `rechargeLevels` per ship — how many discrete recharge steps exist.
- Ship rosters (`games` set per ship) — sensible defaults, edit freely.

## Adding ships / games

Edit **only** `model/Domain.kt`:
- Add a `Ship(...)` to `SHIPS` with its `games` set, `faction`, and `art` id.
  `hasShields` / `hasBeam` auto-show or hide the matching controls.
- Each `Game` references its own `KeyMap` (currently all the shared
  `CLASSIC_KEYMAP`). To diverge one game, set e.g.
  `Game(GameId.TIE, ..., CLASSIC_KEYMAP.copy(fireMode = KeyStep(...)))`.
  The UI and injector never change.

## Adding your cockpit art

The cockpit currently draws a schematic placeholder. In `ui/Screens.kt`,
`CockpitScreen` has a marked `COCKPIT ART PLACEHOLDER` box. Drop your PNGs into
`res/drawable`, layer a base image with `Image(...)`, and swap overlay images
based on `st` (the live `ShipState`) — e.g. a "shields forward" glow when
`st.shield == ShieldConfig.FORWARD`. Tap regions can sit on top as transparent
`HudButton`s. Send the art and I'll wire the layers.

## Known limitations (by design)

- The game never reports its state back, so the app tracks an **assumed** model
  seeded from the known starting config. If you also press keys on a controller/
  keyboard, the assumption can drift. **Reset** re-syncs shields (deterministic)
  and resets the displayed recharge rates.

## Ship commands overlay

The **COMMANDS** button (bottom-left of the cockpit) opens a full-screen overlay
of wingman/comms orders you tap to fire, then CLOSE to return. Grouped as:

- Wingman orders: Attack Target (Shift+A), Ignore Target (Shift+I), Evade
  (Shift+E), Wait/Hold (Shift+W), Go/Proceed (Shift+G), Hyper Home (Shift+H),
  Report In (Shift+R).
- Docking/cargo: Dock Target (Shift+D), Rearm Me — call a resupply craft to dock
  and rearm warheads (Shift+B), Pick Up Object (Shift+P), Enter Hangar (Space).
- Menus: Flight Menu (Tab).

These live in `COMMAND_GROUPS` in `model/Domain.kt` — add/rename/rebind freely.
Note Shift+R is "report in" in the classic games but "release object" in XWA;
confirm in-game.

## Loadout screen

After picking a craft you choose the mission's warhead loadout (this sends no
keys — it only tells the cockpit what the W weapon-cycle should contain):

- Options: No Missiles, Missiles, Adv Missiles, Torpedoes, Adv Torpedoes, Heavy
  Rocket, Space Bomb, Mag Pulse.
- Original X-Wing offers only No Missiles / Missiles / Torpedoes; TIE Fighter
  through XWA offer the full set (per-game list in `Game.warheads`, `Domain.kt`).
- The full menu is offered to every craft, so a mission-adapted TIE Fighter can
  carry missiles.
- The cockpit weapon cycle then shows LASERS + the selected warhead only. With No
  Missiles the weapon button is inert ("LASERS ONLY").
- The Missile Boat (`warheadSlots = 2`) picks two warheads — any combo, including
  doubles. Duplicate types collapse to one entry in the cycle since the app
  doesn't track ammo count.

Loadout is preserved across RESET (it's a mission property, not a flight state).

## Energy management (recharge columns)

Recharge is now vertical bar columns on the far left of the cockpit: Laser,
Shield (if the craft has shields), Engine, and Beam (if the loadout enables it).

- Laser / Shield / Beam: 4 boxes = 5 states (0..4), default 2 filled. Tapping the
  column sends F9 / F10 / F8 and advances one box, wrapping 4 -> 0.
- Engine: DERIVED and display-only (no tap). Pool depends on the craft: 6 for shieldless craft (smaller reactor), 8 for
  shielded, +2 more when beam is on. Beam recharge draws from the pool:
  `engine = pool - laser - shield - beam`. Beam on + beam bar empty = +2 engine
  headroom; charging the beam bar spends it back.
- Energy is CONSERVED. A recharge press can only add a box if the engine has one to
  give. One press does: at max -> wrap to 0 (returns all boxes to engine); else if
  engine > 0 -> +1 (draws from engine); else (engine empty) -> dump to 0 (returns its
  boxes to engine). So e.g. L4 S4 B2 (engine 0), press beam -> B0, engine 2.
- Shieldless craft (e.g. TIE Fighter) hide the Shield column.

Beam presence: the Gunboat always has its beam; the TIE Defender and Missile Boat
get a BEAM ON/OFF toggle on the loadout screen (default OFF — beam is later-game tech).
`Ship.beamOptional` controls which craft show the toggle.

VERIFY in-game: that laser/shield recharge actually has 5 steps and that F9/F10
cycle up and wrap — adjust the box count / cycle in `Domain.kt` if the game differs.

## Ship art spec (shield indicator)

Each craft needs ONE silhouette image (the shield ring/arcs are app-drawn):
- 1024 x 1024 transparent PNG (512 acceptable), same canvas for every craft.
- Ship centered inside an ~80% safe zone; displays ~300-350 px on the Thor's
  1240x1080 bottom screen.
- Filename maps to `Ship.art` in Domain.kt (e.g. cockpit_xwing -> res/drawable).

Shields are now a single toggle: one tap = one S press, cycling equal -> forward
-> rear -> equal (matching the in-game S key). The circular indicator with the
silhouette + state arc replaces the placeholder button once art is supplied.
