# GameSpice Mobile

Touch controls for a first-person game: a drawn stick, drag-to-look, the game's declared buttons, optional gyro look, and opt-in aim assist and auto-fire, each bounded by replicated server cvars. It drives the game's own `GSFpsTouchSampler`; it replaces nothing in it.

Read the family [CLAUDE.md](../../CLAUDE.md) first; this file is only what is specific to mobile.

**The distributable is `addons/gs_mobile/`.** It requires gs-core, gs-ui and gs-player-controller (symlinked and gitignored for development). It is code only: everything is drawn with `_draw`, never a texture.

## Read before

| Read | Before |
| --- | --- |
| [docs/design.md](docs/design.md) | anything: who forwards touches, the server's rules, aim assist, auto-fire, gyro, wiring a game |

## Layout

| Path | What |
| --- | --- |
| `core/gs_mobile_rules.gd` | `GSMobileRules`: what the server allows; the `sv_mobile_*`, `sv_aim_assist*`, `sv_autofire*` cvars; `resolve(prefs, touch)` |
| `core/gs_mobile_prefs.gd` | `GSMobilePrefs`: what the player chose; `to_dict` / `from_dict` |
| `core/gs_mobile_aim_assist.gd` | `GSMobileAimAssist`: cone pick and a capped turn towards it, in the sampler's degrees |
| `core/gs_mobile_autofire.gd` | `GSMobileAutoFire`: when the fire button is down; `ray_target` for a game with no crosshair query |
| `runtime/gs_mobile_gyro.gd` | `GSMobileGyro`: DeviceMotion through `GSWeb`, iOS permission from a gesture |
| `runtime/gs_mobile_buttons.gd` | `GSMobileButtons`: GameSpice UI's `GSTouchButtons`, mirrorable |
| `runtime/gs_mobile_controls.gd` | `GSMobileControls`: the node a game adds |
| `examples/mobile_selftest.tscn` | the suite |

## Running and testing

`sudo -u gamedev /home/gamedev/bin/hv . import`, then `sudo -u gamedev /home/gamedev/bin/hv . scene res://examples/mobile_selftest.tscn`, capped with `timeout`. Read the stderr even when it exits 0. The headless viewport is 64 pixels wide: a check must not depend on the screen size.

## Rules that bite

- The overlay forwards touches itself and sets the sampler's `ignore_fn`; the game guards its own forwarding with `controls.takes(event)`. [design](docs/design.md#who-forwards-a-touch)
- Aim assist and auto-fire change only the sampler's yaw and pitch and a held button bit, never the simulation, hit registration or lag compensation. [design](docs/design.md#aim-assist)
- Auto-fire presses the same bit a finger on Fire sets, so fire modes and cooldowns stay the simulation's. The game's crosshair kind must say FRIEND for teammates and the dead. [design](docs/design.md#auto-fire)
- Everything off-default is the server's call: aim assist is OFF in the addon, auto-fire is OFF in the addon (a game may default it on). [design](docs/design.md#the-servers-rules)
- `GSWeb` callbacks are released in `GSMobileGyro.stop()`; never leave a lambda in a static list. [design](docs/design.md#gyro-look)
- `GSMobileControls` must not name its own class inside its script (it extends a script class).
