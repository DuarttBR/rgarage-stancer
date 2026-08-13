# Ryu Garage — Stancer

Live stance editor for FiveM: track width, camber, suspension height and wheel size,
applied in real time with an orbital camera, saved per plate.

**[▶ Buy on Tebex](TEBEX_LINK_HERE)** · **[📺 Video preview](VIDEO_LINK_HERE)**

> This repository holds the **documentation only**. The resource itself is delivered
> through Tebex.

---

## What it does

A tablet-style panel where every slider applies to the car **as you drag it**, with the
camera orbiting the vehicle at the same time. Save, and the stance comes back with the
car from the garage or after a restart.

## Features

### Stance

- **Track width**, front and rear — how far the wheels sit out
- **Camber**, front and rear
- **Suspension height**
- **Wheel width** and **wheel size**, with stock wheels detected and the tab locked
  until aftermarket wheels are fitted

### Presets

- **5 built-in suspension presets**: Default, Lowered, Sport, 4x4 and Off-Road
- **Unlimited personal presets**, named by the player and saved **per vehicle model**,
  with the current model listed first

### The panel is the player's

- **Two UI themes**: Classic and Modern, fully restyled, not just a colour swap
- **Free accent colour** through a colour picker
- **5 wallpapers** included, plus any custom image URL
- All of it saved per player in `localStorage`

### Orbital camera

Drag to rotate, scroll to zoom, with keyboard fallback. Vehicle controls are locked
while the menu is open, and ESC closes it.

### Optional physical install

Turn `Config.Installation.enabled` on and the stancer becomes something the player has
to fit: walk up with the item, `[E]` opens the hood, `[E]` again starts a progress bar
with a mechanic animation, `[F6]` cancels, and the distance is checked the whole time.

Or leave it off and the item simply opens the panel.

### Tuning shop locations

Fixed points on the map with blips and a `[E] Open Stancer` prompt, each one optionally
restricted to a job or group.

## Requirements

- [`ox_lib`](https://github.com/overextended/ox_lib)
- [`oxmysql`](https://github.com/overextended/oxmysql)

Tables are **created automatically** on first start — no manual import needed.

## Frameworks

| Framework | Guide |
|---|---|
| vRP | [INSTALL_VRP.md](INSTALL_VRP.md) |
| VRPeX | [INSTALL_VRPEX.md](INSTALL_VRPEX.md) |
| QBCore | [INSTALL_QB.md](INSTALL_QB.md) |
| ESX | [INSTALL_ESX.md](INSTALL_ESX.md) |
| Creative Network / V5 | Dedicated bridges — set `Config.Framework` accordingly, otherwise identical to vRP |
| Standalone | `Config.Framework = "standalone"` — no permission checks |

An ACE `admin` always passes, whatever the framework.

## Languages

**10 languages** included: EN, PT-BR, ES, FR, DE, IT, RU, ZH, JA, KO. `Locales.lua`
ships open and editable.

## Built for a live server

- **Per-plate persistence** with a `fake_plate` alias, so a cloned plate keeps writing
  to the original row
- **Stock values captured per model** into their own table — that is what lets Reset
  return the car to exactly how it left the factory
- **Read chain**: statebag → server cache → database → live capture, so most reads never
  touch SQL at all
- **Concurrent requests for the same plate are deduplicated** into a single query
- **Adaptive threads**: the re-apply loop sleeps at ~1 s with nothing modified nearby,
  and the camera loop only runs per-frame while the menu is open
- **Plays nice with `rgarage_suspar`** through a shared statebag, so the two never
  revert each other

## Integration

```lua
-- client
exports['rgarage_stancer']:OpenMenu()
exports['rgarage_stancer']:LoadOnSpawn(vehicle)

-- server, one line from any garage script
exports['rgarage_stancer']:loadStanceOnSpawn(netId, plate, modelName)
```

ox_inventory users get a plug-and-play export for the item:
`server = { export = 'rgarage_stancer.useStancerItem' }`

## What is open

`config.lua`, `Locales.lua`, the SQL file, every framework bridge and the whole UI ship
unencrypted. Core logic is escrow protected.

## Support

Found a bug or need help with the install? Open an issue in this repository.
