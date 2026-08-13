# rgarage_stancer — Installation on ESX

Covers both the **ESX built-in inventory** and **ox_inventory**.

---

## 1. Dependencies

Start these **before** `rgarage_stancer` in `server.cfg`:

- `es_extended`
- `oxmysql`
- `ox_lib`

```cfg
ensure es_extended
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
Config.Framework = "esx"
Config.Language  = "en"   -- en, pt_br, es, fr, de, it, ru, zh, ja, ko
```

## 4. fxmanifest.lua — required on ESX

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

On ESX the key is matched against the permission group (`getGroup()`) **or** the job name — either one passing is enough:

```lua
groups = { ["mechanic"] = 0, ["admin"] = 0 }
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

### ESX built-in inventory

Add the item to your `items` table, then in your own server file:

```lua
ESX = exports['es_extended']:getSharedObject()

ESX.RegisterUsableItem('stancer', function(src)
    TriggerEvent("rgarage_stancer:server:useItem", src, 'stancer')
end)
```

## 7. Notify and progress bar

```lua
Config.Notify   = { enabled = true, type = "ox_lib" }
Config.Progress = { enabled = true, type = "ox_lib" }
```

`ox_lib` is the right default on ESX for both.

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
`@vrp/lib/utils.lua` is still active in the fxmanifest — it must be commented out on ESX. With `Installation.enabled = true` the car also has to have the stancer fitted first.

**Permission always denied**
The key has to be the job name or the permission group, spelled as your server returns it.

**Reset does not return the original stance**
The stock row for that model is captured the first time the panel is opened on it. If
the car was already modified by another resource at that moment, that is what got
stored as "stock".

**Nothing saves**
`oxmysql` has to start before the resource. Both tables are created automatically —
check the console on first start.
