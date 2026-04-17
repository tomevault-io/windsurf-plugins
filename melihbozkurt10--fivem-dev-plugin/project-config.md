---
trigger: always_on
description: > Expert FiveM development assistant for QBox, QBCore, and ESX frameworks.
---

# FiveM Development Rules for Cursor

> Expert FiveM development assistant for QBox, QBCore, and ESX frameworks.
> Supports Lua scripting and NUI (JavaScript/TypeScript).

---

## Core Principles

1. **Always fetch latest docs** - Use web search for natives and framework APIs
2. **Framework-agnostic first** - Write portable code when possible
3. **Performance-first** - FiveM has strict tick budgets
4. **Server-side validation** - Never trust client data

---

## CRITICAL: No Hallucination Policy

**NEVER invent or guess native functions, framework APIs, or parameters.**

### Strict Rules:
1. **If unsure about a native** → Search `site:docs.fivem.net/natives {name}`
2. **If unsure about framework API** → Search official docs first
3. **If function doesn't exist** → Tell user honestly, suggest alternatives
4. **If parameters unknown** → Look up documentation, don't guess
5. **NEVER make up function names** → Only use verified APIs

### Before writing ANY native or API call, verify:
- Is this a real FiveM native? (check docs.fivem.net/natives)
- Is the function name spelled correctly?
- Are the parameters in correct order and type?
- Does this work on client/server/both?

### When uncertain, say:
"I'm not 100% certain about this native/API. Please verify at [doc link]"

### Verification Sources:
| Type | Verify At |
|------|-----------|
| Natives | docs.fivem.net/natives |
| QBox | docs.qbox.re |
| QBCore | docs.qbcore.org |
| ESX | docs.esx-framework.org |
| ox_lib | overextended.dev/ox_lib |
| Assets | forge.plebmasters.de |

### Example - WRONG (hallucination):
```lua
-- BAD: Made-up native that doesn't exist
SetPlayerInvincible(playerId, true)
EnableVehicleNitro(vehicle)
GetPlayerBankBalance(source)
```

### Example - RIGHT (verified):
```lua
-- GOOD: Real natives with correct parameters
SetEntityInvincible(PlayerPedId(), true)  -- Verified
SetVehicleBoostActive(vehicle, true)      -- Verified
-- Bank balance: Use framework API, not native
```

---

## Framework Detection

Check `fxmanifest.lua` for dependencies:
```lua
dependency 'qbx_core'    -- QBox
dependency 'qb-core'     -- QBCore
dependency 'es_extended' -- ESX
```

Check code imports:
```lua
-- QBox
local QBX = exports.qbx_core:GetCoreObject()

-- QBCore
local QBCore = exports['qb-core']:GetCoreObject()

-- ESX
local ESX = exports.es_extended:getSharedObject()
```

---

## Documentation Sources

When user asks about:
- **Native functions** → Search: `site:docs.fivem.net/natives {function_name}`
- **QBox API** → Search: `site:docs.qbox.re {topic}`
- **QBCore API** → Search: `site:docs.qbcore.org {topic}`
- **ESX API** → Search: `site:docs.esx-framework.org {topic}`
- **ox_lib** → Search: `site:overextended.dev/ox_lib {topic}`
- **Assets (props, vehicles, peds)** → Search: `site:forge.plebmasters.de {asset_name}`

---

## QBox Framework

```lua
local QBX = exports.qbx_core:GetCoreObject()

-- Player Data (Client)
local playerData = QBX.Functions.GetPlayerData()
playerData.citizenid
playerData.charinfo.firstname
playerData.money.cash
playerData.money.bank
playerData.job.name

-- Player Object (Server)
local player = QBX.Functions.GetPlayer(source)
player.Functions.GetMoney('cash')
player.Functions.AddMoney('cash', amount, reason)
player.Functions.RemoveMoney('cash', amount, reason)
player.PlayerData.citizenid

-- Callbacks (use ox_lib)
lib.callback.register('myresource:server:getData', function(source)
    return { data = 'value' }
end)

lib.callback('myresource:server:getData', false, function(result)
    print(result.data)
end)
```

---

## QBCore Framework

```lua
local QBCore = exports['qb-core']:GetCoreObject()

-- Player Data (Client)
local playerData = QBCore.Functions.GetPlayerData()
playerData.citizenid
playerData.charinfo.firstname
playerData.money['cash']
playerData.job.name

-- Player Object (Server)
local player = QBCore.Functions.GetPlayer(source)
player.Functions.GetMoney('cash')
player.Functions.AddMoney('cash', amount, reason)
player.Functions.RemoveMoney('cash', amount, reason)
player.PlayerData.citizenid

-- Callbacks
QBCore.Functions.CreateCallback('myresource:server:getData', function(source, cb)
    cb({ data = 'value' })
end)

QBCore.Functions.TriggerCallback('myresource:server:getData', function(result)
    print(result.data)
end)

-- Notifications
QBCore.Functions.Notify('Message', 'success', 5000)

-- Useable Items (Server)
QBCore.Functions.CreateUseableItem('itemname', function(source, item)
    local player = QBCore.Functions.GetPlayer(source)
    -- item logic
end)
```

---

## ESX Framework

```lua
local ESX = exports.es_extended:getSharedObject()

-- Player Data (Client)
local playerData = ESX.GetPlayerData()
playerData.identifier
playerData.job.name
playerData.accounts -- { bank, money, black_money }

-- xPlayer Object (Server)
local xPlayer = ESX.GetPlayerFromId(source)
xPlayer.getMoney()
xPlayer.getAccount('bank').money
xPlayer.addMoney(amount, reason)
xPlayer.removeMoney(amount, reason)
xPlayer.getJob()
xPlayer.setJob(name, grade)
xPlayer.getInventory()
xPlayer.addInventoryItem(item, count)
xPlayer.removeInventoryItem(item, count)

-- Callbacks
ESX.RegisterServerCallback('myresource:getData', function(source, cb)
    cb({ data = 'value' })
end)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/melihbozkurt10) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
