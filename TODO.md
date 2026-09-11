X-Wing Cockpit — Status & Roadmap
1.0 — shipped

X-Wing Cockpit 1.0 is feature-complete and validated on AYN Thor hardware. See the README for the full feature list. In short, the following are done and working in-flight: Shizuku key injection (including true Shift-chord commands), the non-focusable window that keeps the game focused, press-down touch handling, the full cockpit control set for all four games, single-key energy transfer, correct shield-cycle direction, per-faction laser colors, the color-coded weapon button, and the mission-goal system across X-Wing (100), TIE Fighter (76), and X-Wing Alliance (53) with hidden bonus objectives.

Confirmed on hardware
* Game keeps focus and controller input while the panel is used.
* Single-key injection (shields, weapons, throttle, recharge).
* Shift commands (wingman orders, dock/rearm) via the real-keyboard injection path.
* Energy transfer (' / ;) moves energy correctly in both directions.
* Shield cycle order matches the game (EQUAL → REAR → FORWARD).
* Quick/light presses register reliably (press-down firing + ~40 ms key hold).

Verify as you play the other games
Most keybindings are confirmed, but a few are worth a quick check in each specific title as you get hours in, since they can differ per game:
* Ship targeting keys (Next/Prev Target T/Y, Nearest Fighter R) and Engage Hyperdrive (H) / Cockpit Display (I).
* Report In (Shift+R) is "release object" in X-Wing Alliance rather than "report in."
* Throttle punctuation keys (] [ \ /) are the most layout-sensitive through Winlator. Anything that turns out different is a one-line edit in model/Domain.kt (CLASSIC_KEYMAP, or a per-game .copy(...)).

Possible future enhancements (nice-to-have, not planned)
* Swap the fire-mode warhead dots for actual warhead icons.
* Remember the last-used game/ship on launch.
* Optional per-game keymap overrides if more divergences surface.
* Add Critical campaign mission ships for XWA.  Azzameen family cargo transports, and Millennium Falcon.

Intentionally not doing
* Hardware-button shortcut to the goals screen. The non-focusable window (which is what keeps the game focused) can't observe controller/keyboard keys, so this would require an accessibility service or Shizuku input-monitoring plus a per-device button-learn step — too much setup for the benefit. Use the on-screen Mission Goals command instead.
