# Real Room Temperature 2.0

Integration documentation for mod authors. [Real Room Temperature 2.0 on Steam Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3741066319)

## Lua cold-retention hook

With **Body Temperature Rework** enabled, your mod can supply a per-player reduction in heat lost to colder air. Real Room Temperature 2.0 applies the value when its Lua body model runs in a room or vehicle cabin. Your mod tracks the player’s reserve and decides how it builds up and fades.

| Game mode | Where to register |
| --- | --- |
| Singleplayer | In the local singleplayer Lua environment. |
| Multiplayer | On the server. Track the reserve and register the callback server-side. |

### Register a provider

This example assumes your mod declares Real Room Temperature 2.0 as a dependency. Use a unique provider ID, such as your mod ID.

```lua
local retention = require("RRH2/RRH2ColdRetention")

retention.registerColdRetentionProvider("YourModID", function(player)
    return 0.50
end)
```

Replace `0.50` with a lookup of the current reserve for the supplied player. Keep the callback fast and read-only. Real Room Temperature 2.0 samples it once per eligible prediction batch.

### Return a fraction from 0.0 to 1.0

| Return value | Effect on heat loss to colder air |
| --- | --- |
| `nil` or `0.0` | No reduction. |
| `0.10` | Reduces it by 10%. |
| `0.50` | Reduces it by 50%. |
| `1.0` | Prevents it entirely. |

This fraction scales heat loss to colder air. Core temperature still depends on the rest of the body model. Real Room Temperature 2.0 uses the highest valid contribution if multiple mods register providers. Finite values above `1.0` are treated as `1.0`. Negative or non-finite values, other return types, and callback errors give no contribution.

Outdoors on foot, the base game handles body temperature. Native body-temperature delivery and bath or shower calculations also follow their usual paths. Your mod can keep tracking the reserve and supply it when the Lua air model runs again.

### Remove or replace a provider

Call this when disabling the integration.

```lua
retention.unregisterColdRetentionProvider("YourModID")
```

Registering again with the same ID replaces the previous callback. If Real Room Temperature 2.0 is an optional dependency, wait until mods have loaded and check for `RealRoomHeat2.ColdRetention` before registering.
