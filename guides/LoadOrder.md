# TLDR:
Load order does almost nothing and trying to fix mod conflicts with it is a waste of time. Map load order is separate and does matter.
# Load order in Project Zomboid
This document details the effects of mod load order in Project Zomboid.
## File overrides
The purpose of mod load order is to control file overrides.
When the game tries to load a file, if multiple mods (including vanilla) have a matching file, only the file from the mod latest in the load order is actually loaded.

The only exception to this rule is that certain file types don't use overrides at all, and the file from every mod is loaded. A bug exists where these may still emit file override debug messages, but this is harmless.
These systems do load their files in mod load order order, but this is deliberately restricted to systems where this does not really matter.
# Yes, that's it
That's all changing load order does. Load order is not nearly as important in Project Zomboid as it in many other games, and changing load order will rarely if ever fix conflicts between mods.
A conflicting file between mods in most cases just means they can't work together at all - all changing the order will do is choose which mod to break.
This makes sense when you consider that the vanilla game doesn't even have features to support changing load order - it's not expected or intended to do anything.

The only situation where load order will consistently help is with mods that change the game's textures or models. Moving a mod later in the order will prioritise that mod's textures and models.
## Map load order
Maps are not affected by regular mod load order - they use a separate order. The only real difference is that it works in map cells (300x300 areas), not files, and **earlier** maps are prioritised over later maps.
# For developers
For Lua and scripts, file overrides are generally frowned upon for multiple reasons, so just don't use them - in almost all cases you can achieve the same effect without overriding the file.
This way you don't need to tell people to change their load order, which will inevitably lead to people doing it wrong or ignoring it and complaining.
You are also more compatible with other mods as two mods that override the same file are literally guaranteed to conflict to some degree, as only one of the overrides will actually run.

In most cases you should also be able to write your code in a manner that load order does not matter, but this is not always possible.
Lua and script files are loaded alphabetically (including folder names), so if you need it to load as early or late as possible, just name it so that it will. Mod load order has absolutely no effect.
Keep in mind that vanilla lua and scripts always load before modded files (For lua, this is separated in each folder - e.g. vanilla shared, then modded shared).
For scripts, files prefixed with ``template_`` load before other files.

To avoid overriding lua scripts, most mods will store their functions as globals which you can simply redefine in a file that loads later.
If the file uses a [modular structure](guides/Modules.md) then you can do the same by importing and modifying their module.

If the file has locals you cannot access or executes code before you can get to it, you can take the nuclear option.
Name your file so that it will load earlier (alphabetically), and replace any functions you want to prevent it from calling with a do-nothing function ``function() end``.
Store the originals before you do this so you can put them back afterwards and not break the function forever. Require the file, and then put the original functions back. An example:
```lua
----------------------------------------------------------------
-- TargetFile.lua
----------------------------------------------------------------

-- oh nooo a local function i can't edit :(
local function test()
    print("test")
end

Events.OnGameStart.Add(test)

----------------------------------------------------------------
-- AAMyFile.lua (named so that it loads first)
----------------------------------------------------------------

-- save the original function
local old_add = Events.OnGameStart.Add
-- replace it with a do-nothing function
Events.OnGameStart.Add = function() end
-- force the target file to run now
require("TargetFile.lua")
-- put the original function back
Events.OnGameStart.Add = old_add
```
"test" will no longer be printed at game start, because it was never added to the event, because the Add function was replaced with a dummy when the file actually ran.
