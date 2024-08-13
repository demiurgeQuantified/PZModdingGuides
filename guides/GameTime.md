# Game Time and deltas
Note: In Project Zomboid, the terms 'frame' and 'tick' generally refer to the same thing.
## What is a time delta?
'Delta' is a mathematics term for 'change in'. In a programming context, the time delta is how much time has passed since the last tick (the change in time).
These are useful to make sure effects are applied consistently between different frame rates.
For example, if a mod healed the player by 1 health every frame, at 60 frames per second the player would gain 60 health per second, but at 30fps they would only gain 30 health per second.
By multiplying the effect by the time delta, it can be ensured that the player will gain health at exactly the same rate at any framerate.
This is extremely important to ensure mods behave the same on different hardware.
### When to use time deltas
- When an effect is being applied over time, in a way that is tied to the framerate/tickrate (OnTick, OnPlayerUpdate, etc)
- When a random chance is being rolled repeatedly, in a way that is tied to the framerate/tickrate
- Generally whenever a value is being updated every tick and is expected to always change at the same speed
### When NOT to use time deltas
If you use the time delta inappropriately, you will actually end up tying things to framerate instead of untying them.
- When an effect is applied all at once
- When an effect is being applied over time, but in a way that is not tied to framerate (EveryMinute, EveryTenMinutes, etc)
## Time deltas in Project Zomboid
Project Zomboid introduces extra complexity with time deltas:
- In singleplayer, the game speed can be multiplied, and so game world effects that are tied to time should be multiplied, even though the amount of real time since the last frame is not actually increasing.
- There is in-game time, and the rate at which this in-game time passes should scale many effects.
  - For example, characters should always take a certain amount of in-game hours to get hungry, independent of how much real time that actually is

Luckily, you don't need to adjust for these manually - the [GameTime](https://projectzomboid.com/modding/zombie/GameTime.html) object provides deltas in multiple formats for these uses.

## An example of delta usage
This code aims to passively heal the player by 1 health every real life second. 
```lua
local function healingOnTick()
    local player = getSpecificPlayer(0)
    -- if the player doesn't exist (e.g. dead), stop execution
    if not player then return end

    -- Divide the target amount of health by 60, as this code runs every frame, and we're playing at 60 frames per second
    player:getBodyDamage():AddGeneralHealth(1 / 60)
end
Events.OnTick.Add(healingOnTick)
```
The problem with this function is that if the game slows down or speeds up, due to lag, a weaker machine, or an uncapped framerate, ticks will be processed faster so there will be more ticks per second. A smarter approach would be as follows:
```lua
local function healingOnTick()
    local player = getSpecificPlayer(0)
    if not player then return end

    -- Multiply the target amount of health by the unmodded multiplier, making it heal at a consistent rate even if the game lags or runs too fast
    player:getBodyDamage():AddGeneralHealth(1 * getGameTime():getUnmoddedMultiplier())
end
Events.OnTick.Add(healingOnTick)
```

## The deltas, what they mean, and when to use them
When unspecified, a delta adds up to 1 per second. This means that you should multiply the delta by the amount you want it to increase every real world second. If it scales with day length, this is multiplied for longer days (1x for thirty minute days, 2x for one hour days, 4x for two hour days, etc). If it scales with game speed, it is multiplied by the game speed (1x, 5x, 20x and 40x).
### [getRealworldSecondsSinceLastUpdate()](https://projectzomboid.com/modding/zombie/GameTime.html#getRealworldSecondsSinceLastUpdate())
This is the 'raw' time delta. It scales with nothing.
Use this for effects that aren't related to game time at all (e.g. ui visual effects).
### [getMultipliedSecondsSinceLastUpdate()](https://projectzomboid.com/modding/zombie/GameTime.html#getMultipliedSecondsSinceLastUpdate())
This is the time delta from **getRealworldSecondsSinceLastUpdate()**, but scales with the game speed multiplier.
Use this for effects that are only tied to game speed but not day length.
### [getMultiplier()](https://projectzomboid.com/modding/zombie/GameTime.html#getMultiplier())
This is the game speed delta. It scales with the game speed and the day length.
For legacy reasons, this delta actually adds up to 0.8 a second, not 1.
This means that when using it you should expect only 80% of the base amount being multiplied to be applied per second, not 100% as with other most other deltas.
Use this when you want your effect to scale by game world time, and be slower/faster depending on the day length.
### [getInvMultiplier()](https://projectzomboid.com/modding/zombie/GameTime.html#getInvMultiplier())
This is the inverse of **getMultiplier()** - it returns 1 divided by the result of **getMultiplier()**.
This is useful for RNG functions that are called constantly - multiply the chance by this to keep the chance consistent at any framerate.
### [getTimeDelta()](https://projectzomboid.com/modding/zombie/GameTime.html#getTimeDelta())
This is the game speed delta, similar to the one from **getMultiplier()**.
The difference is that this one adds up to 1 every minute, not 0.8 every second.
### [getUnmoddedMultiplier()](https://projectzomboid.com/modding/zombie/GameTime.html#getUnmoddedMultiplier())
This is similar to the regular **getMultiplier()**, but doesn't scale with day length.
This one adds up to 1 every second, not 0.8. Use this when you want your effect to scale with game speedup, but shouldn't be tied to the day length.
### [getGameWorldSecondsSinceLastUpdate()](https://projectzomboid.com/modding/zombie/GameTime.html#getGameWorldSecondsSinceLastUpdate())
The number of game time seconds that have passed since the last frame.
Keep in mind that at default day length (1 hour), one in-game second is roughly 400 milliseconds, so this value will be very high even at 1x speed.
### [getServerMultiplier()](https://projectzomboid.com/modding/zombie/GameTime.html#getServerMultiplier())
Oddly, this multiplier doesn't actually use a delta - it is always based on the target framerate, not the real framerate.
This scales by the game speed, and adds up to 8 every second.
The usage for this is unknown, and the vanilla game doesn't even use it.
### [getTrueMultiplier()](https://projectzomboid.com/modding/zombie/GameTime.html#getTrueMultiplier())
Returns the current game speed multiplier (Speed up through the UI or while sleeping)

## Cheat sheet
| Name | Game Speed | Day Length | Total (1x speed, 30 minute days) |
| --- | --- | --- | --- |
| getRealworldSecondsSinceLastUpdate() | No | No | 1/s |
| getMultipliedSecondsSinceLastUpdate() | Yes | No | 1/s |
| getUnmoddedMultiplier() | Yes | No | 1/s |
| getMultiplier() | Yes | Yes | 0.8/s |
| getInvMultiplier() | Yes | Yes | Inverse |
| getTimeDelta() | Yes | Yes | 1/m |
| getGameWorldSecondsSinceLastUpdate() | Yes | Yes | 1/game second |

Note: It seems that getUnmoddedMultiplier() and getMultipliedSecondsSinceLastUpdate() are completely redundant with each other.

## Notes on performance
An optimisation that can be made to any mod that requires the use of multipliers (or GameTime in general) is to cache the GameTime object. This object never changes during gameplay, so it can safely be cached when the game loads.
```lua
-- Place this in your file somewhere above your GameTime calls
local gameTime
Events.OnGameTimeLoaded.Add(function()
    gameTime = GameTime.getInstance()
end)
```
Now whenever you call on GameTime, you can use the cached object instead of getGameTime()/GameTime.getInstance()
```lua
-- Replace method calls like these these
getGameTime():getMultiplier()
-- With method calls like these
gameTime:getMultiplier()
```
This saves significant amount of performance, as you don't need to call a java method every time you access GameTime. Zomboid's lua implementation has a non-insignificant performance penalty whenever a java method is called. This way you effectively half that overhead on any GameTime call.

A further optimisation that can be made is that if you're using the same delta multiple times in the same tick, you can cache it once and reuse the value. The value never changes within a given tick. This can have significant performance benefit as on top of the java method overhead, the deltas are recalculated every method call, even though they don't change.
```lua
-- Whenever you use the same multiplier multiple times like this
local foo = 5 * gameTime:getMultiplier()
local bar = 15 * gameTime:getMultiplier()
-- Cache the value once instead
local multiplier = gameTime:getMultiplier()
local foo = 5 * multiplier
local bar = 15 * multiplier
```

[Return](../README.md)
