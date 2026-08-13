# rgarage_stancer — Installation on VRPeX

---

## 1. Dependencies

Start these **before** `rgarage_stancer` in `server.cfg`:

- `vrp` (VRPeX)
- `oxmysql`
- `ox_lib`

```cfg
ensure vrp
ensure oxmysql
ensure ox_lib
ensure rgarage_stancer
```

## 2. Database

**Nothing to run by hand.** Both tables are created on first start:

- `rgarage_stancer_configs` — the saved stance of each vehicle, by plate
- `rgarage_stancer_stock` — the factory values of each model, captured the first time
  someone opens the panel on it. This is what makes Reset return the exact original.

The `.sql` file in the resource is there for reference.

## 3. config.lua

```lua
Config.Framework = "vrpex"
Config.Language  = "pt_br"   -- en, pt_br, es, fr, de, it, ru, zh, ja, ko
```

## 4. fxmanifest.lua — required for VRPeX

This line must be **uncommented** in `shared_scripts`:

```lua
'@vrp/lib/utils.lua'
```

Without it the bridge errors out with *"vRP não está disponível"* and the resource does
not start.

> **Linux is case sensitive.** Some builds ship `lib/Utils.lua` with a capital U. Check
> the real filename in your `vrp/lib/` folder and match it exactly.

## 5. Command and permissions

```lua
Config.Commands = {
    enabled = true,
    name    = "stancer",    -- /stancer
    groups  = {}            -- empty = anyone can use
}
```

On VRPeX either a group or a permission name works:

```lua
groups = { ["Mechanic"] = 0, ["Admin"] = 0 }
```

There is also `/reloadstance`, with no permission check, which re-applies the saved
stance to the vehicle you are in. Useful for support.

An ACE `admin` always passes, whatever the framework.

## 6. The item (optional)

Two modes, controlled by `Config.Installation.enabled`:

- **`false`** — using the item just opens the panel
- **`true`** — the item **fits** the stancer to the car: stand outside it, `[E]` opens
  the hood, `[E]` again runs the progress bar with a mechanic animation, `[F6]` cancels.
  Only then does the vehicle accept the panel.

Add the item to your itemlist, and fire this from its use handler:

```lua
TriggerServerEvent("rgarage_stancer:server:useItem", source, "stancer")
```

## 7. Notify and progress bar

```lua
Config.Notify   = { enabled = true, type = "ryu" }
Config.Progress = { enabled = true, type = "event" }
```

`"ryu"` fires `ryu_hud:notification`; `"event"` fires `TriggerEvent("Progress", duration)`, the usual vRP pattern.

The config file already carries ready-made `custom` examples for okokNotify, pNotify and
a generic `Notify` event.

## 8. Worth tuning before going live

```lua
Config.AllowedVehicleClasses   -- 23 classes; bikes, boats and planes are off by default
Config.Limits                  -- camber, track width, suspension and wheel ranges
Config.BlipLocations           -- fixed tuning shops, each with its own permission
Config.HideResetButton         -- hide Reset from players
Config.SaveMode                -- "database" or "statebag"
```

---

## Troubleshooting

**The panel does not open**
`Config.Framework` is not `"vrpex"`, or `@vrp/lib/utils.lua` is still commented. With `Installation.enabled = true` the car also has to have the stancer fitted first.

**Permission always denied**
The key in `groups` is not a real group or permission on your base. Case matters.

**Reset does not return the original stance**
The stock row for that model is captured the first time the panel is opened on it. If
the car was already modified by another resource at that moment, that is what got
stored as "stock".

**Nothing saves**
`oxmysql` has to start before the resource. Both tables are created automatically —
check the console on first start.
