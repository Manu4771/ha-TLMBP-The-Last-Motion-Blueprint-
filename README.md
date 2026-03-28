# TLMBP – The Last Motion Blueprint

> The only motion automation blueprint you'll ever need for Home Assistant.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmanu4771%2Ftlmbp%2Fblob%2Fmain%2Fblueprints%2Ftlmbp.yaml)

---

## Why TLMBP?

Most motion blueprints do one thing: turn lights on and off. That works fine until you want more control — multiple sensors, time-based scenes, brightness-aware automation, and most importantly: **the automation should know when you've taken manual control and step back**.

TLMBP solves all of that in a single, flexible blueprint that works for every room in your home.

---

## Features

- **Multiple motion/presence sensors** — combine as many as you need per room
- **Time-based scene selection** — Morning, Day, Evening, Night with configurable start times
- **Illuminance cutoff** — only trigger when it's actually dark enough
- **Configurable trigger delay** — compensates for slow Zigbee/Hue sensor updates
- **Automation Blocker** — completely suppress the automation (e.g. someone is sleeping)
- **Manual Override Detection** — the blueprint detects when you or Alexa have taken control and will not override your state or turn off your lights

---

## How Manual Override Detection Works

This is TLMBP's key feature and what sets it apart from other motion blueprints.

Home Assistant stores a `context` on every state change — metadata about what caused the change. TLMBP uses `context.parent_id` to distinguish between automation-controlled and manually-controlled states:

| Who changed the light last | `parent_id` | Automation acts? |
|---|---|---|
| This automation | set (internal HA ID) | ✅ Yes |
| Manual (HA UI) | `None` | ❌ Blocked |
| Alexa via Nabu Casa | `None` | ❌ Blocked |
| Light is `off` | — | ✅ Always turns on |

**What this means in practice:**

- You walk through the dining room → automation turns on 5 spots
- You decide to switch on all 11 spots via Alexa
- Someone walks through again → automation detects manual override → **lights stay on, nothing changes**

- Motion triggers living room evening scene (dimmed, 40%)
- You ask Alexa for full brightness (100%)
- Motion triggers again → automation detects override → **scene is not re-applied, lights stay at 100%**

The override clears automatically as soon as the light is turned off completely.

---

## Inputs

### Trigger
| Input | Description | Default |
|---|---|---|
| `motion_entity` | One or more motion/presence sensors | required |
| `motion_delay` | Delay in ms before acting (compensates slow sensors) | `500` ms |
| `light_target` | Target lights, switches, or actors | required |

### Illuminance
| Input | Description | Default |
|---|---|---|
| `illuminance_sensor` | Optional lux sensor | — |
| `illuminance_cutoff_value` | Only trigger at or below this lux value | `999999` |

### Timing
| Input | Description | Default |
|---|---|---|
| `no_motion_wait` | Seconds to wait after last motion before turning off | `120` s |

### Automation Blocker
| Input | Description | Default |
|---|---|---|
| `enable_automation_blocker` | Enable/disable the blocker feature | `true` |
| `automation_blocker` | Entity to use as blocker (e.g. input_boolean) | — |
| `automation_blocker_boolean` | `true` = block when ON / `false` = block when OFF | `false` |

The blocker **completely suppresses the entire automation** — both turn-on and turn-off.
Use case: someone is sleeping in the room, no motion reaction at all.

### Manual Override – Context Tracking
| Input | Description | Default |
|---|---|---|
| `context_entity` | Single light entity used as reference for override detection | — |

Pick one light entity that is part of your target group. TLMBP uses it as a "witness" to detect whether the light was last changed by the automation or by a user/Alexa.

> **Note:** This uses `context.parent_id`, not `context.user_id`.
> Alexa via Nabu Casa does not set a `user_id` but also does not set a `parent_id`,
> which makes `parent_id` the reliable indicator for automation control.

### Scenes
| Input | Description | Default |
|---|---|---|
| `use_scenes` | Enable time-based scene selection | `true` |
| `scene_morning` + `time_scene_morning` | Scene and start time for morning | `06:00` |
| `scene_day` + `time_scene_day` | Scene and start time for day | `09:00` |
| `scene_evening` + `time_scene_evening` | Scene and start time for evening | `18:00` |
| `scene_night` + `time_scene_night` | Scene and start time for night | `22:00` |

If no scene is configured for the current time slot, or `use_scenes` is disabled, the automation falls back to a direct `turn_on` of the target.

---

## Known Limitations

- **`context_entity` must be a single light entity** — not a group, area, or target. Pick any one entity from your target group as a reference.
- **Context is lost after HA restart** — after a restart, `parent_id` will be `None` until the automation next controls the light. In practice this is harmless: the first motion event will re-establish automation control.
- **Other automations setting `parent_id`** — if another automation changes the same light, it will also set a `parent_id`. TLMBP will then not block in that case. This is intentional behaviour.

---

## Changelog

### v1.0.0
- Initial release
- Context tracking via `parent_id` (compatible with Alexa/Nabu Casa)
- Automation Blocker (global, both turn-on and turn-off)
- Time-based scene selection with midnight-spanning night block
- Illuminance cutoff support
- Multi-sensor support

---

## License

MIT License — see [LICENSE](LICENSE)

---

## Credits

Based on [YAMA V10](https://gist.github.com/networkingcat/a1876d7e706e07c8bdcf974113940fb8) by networkingcat.
Extended and redesigned with manual override detection, context tracking, and improved scene handling.
