# rgarage_stancer — Installation on QBCore

Covers both **qb-inventory** and **ox_inventory**. The bridge accepts either
`qb-core` or `qbcore` as the core resource name.

---

## 1. Dependencies

Start these **before** `rgarage_stancer` in `server.cfg`:

- `qb-core`
- `oxmysql`
- `ox_lib`

```cfg
ensure qb-core
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
Config.Framework = "qb"
Config.Language  = "en"   -- en, pt_br, es, fr, de, it, ru, zh, ja, ko
```

## 4. fxmanifest.lua — required on QBCore

This line must stay **commented out** in `shared_scripts`:

```lua
-- '@vrp/lib/utils.lua'
```

There is no `vrp` resource on your server, so the path is invalid and the resource will
not start. It ships commented by default.

## 5. Command and permissions

```lua
Config.Commands = {
    enabled = true,
    name    = "stancer",    -- /stancer
    groups  = {}            -- empty = anyone can use
}
```

On QBCore the key is a **job** or a **gang** name, and the value is the minimum grade:

```lua
groups = { ["mechanic"] = 0, ["police"] = 2 }
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

### ox_inventory

In `ox_inventory/data/items.lua`:

```lua
['stancer'] = {
    label = 'Stancer Kit',
    weight = 500,
    stack = true,
    close = true,
    description = 'Adjust the stance of your vehicle',
    server = { export = 'rgarage_stancer.useStancerItem' }
},
```

Plug and play — the export already exists.

### qb-inventory

Add the item to `qb-core/shared/items.lua`, then in your own server file:

```lua
local QBCore = exports['qb-core']:GetCoreObject()

QBCore.Functions.CreateUseableItem('stancer', function(src)
    TriggerEvent("rgarage_stancer:server:useItem", src, 'stancer')
end)
```

## 7. Notify and progress bar

```lua
Config.Notify   = { enabled = true, type = "ox_lib" }
Config.Progress = { enabled = true, type = "ox_lib" }
```

`ox_lib` is the right default on QB for both.

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
`@vrp/lib/utils.lua` is still active in the fxmanifest — it must be commented out on QBCore. With `Installation.enabled = true` the car also has to have the stancer fitted first.

**Permission always denied**
The key in `groups` is a job or gang name, not a QB permission level.

**Reset does not return the original stance**
The stock row for that model is captured the first time the panel is opened on it. If
the car was already modified by another resource at that moment, that is what got
stored as "stock".

**Nothing saves**
`oxmysql` has to start before the resource. Both tables are created automatically —
check the console on first start.
