# Optimisation
A collection of general advice for improving the performance of Lua code.
## Caching
It is much faster to save objects you plan to access multiple times in a function into a variable than to repeatedly call their getter function:
```lua
-- slower:
getPlayer():Say("I found some fruit!")
getPlayer():getInventory():AddItem("Base.Apple")
getPlayer():getInventory():AddItem("Base.Banana")
-- faster:
local player = getPlayer()
player:Say("I found some fruit!")
local inventory = player:getInventory()
inventory:AddItem("Base.Apple")
inventory:AddItem("Base.Banana")
```
It also often makes your code a lot easier to read.

Additionally, a lot of core game objects don't change often if at all, so calling their getters every time you access them is pretty wasteful, especially if you're accessing them every frame:
```lua
-- slower:
Events.OnTick.Add(function()
    local zombieList = getCell():getZombieList()
    print(zombieList:size() .. "zombies currently loaded")
end)

-- faster:
local zombieList
-- Event fired when the cell loads
Events.OnPostMapLoad.Add(function(cell)
    zombieList = cell:getZombieList()
end)

Events.OnTick.Add(function()
    print(zombieList:size() .. "zombies currently loaded")
end)
```
A caveat to this is some getter methods may actually return copies rather than references to the originals, so they won't be updated when the original object is. However this is rarely done in PZ's codebase.
## Just don't use globals
Globals are really slow to access. Don't create new globals (see [Modules](Modules.md) for guidance on writing larger projects without using globals) and if you're going to use an existing global more than once (especially in a loop), cache it in a local first:
```lua
-- slower:
GlobalFunc()
GlobalFunc()
-- faster:
local GlobalFunc = GlobalFunc
GlobalFunc()
GlobalFunc()
```
## Functions suck
Functions have really heavy overhead in Kahlua. While functions are absolutely vital for good code maintainability, try not to use functions where they're really unnecessary. If a piece of code is **really** performance sensitive, you might want to avoid using them entirely.

An example where functions are generally not worthwhile is Lua's ``math`` library: many of its functions are equivalent to a single line of code, and are so inexpensive that function overhead ends up making up nearly the entire cost of the function. Writing these out where needed rather than calling on the functions saves almost the entire cost.
## Java methods really suck
There is a much heavier overhead to calling Java methods. Try to call as few Java methods as possible - if you can do the same thing in pure Lua it is probably faster. Keep in mind that the **number of Java calls from lua** is what you need to reduce, not how much work the Java actually does - Java is generally faster than the equivalent Lua, it is only calling it that is slow.
## pairs sucks (and ipairs really sucks)
Iterators like ``pairs`` and ``ipairs`` are slow (especially ``ipairs``). Table accesses like ``t[k]`` are very fast. You can leverage this fact to loop through array-like tables much faster:
```lua
-- slower:
for i, v in ipairs(t) do

end

-- MUCH faster:
for i = 1, #t do
    local v = t[i]
    ...
end
```
You should **never use ipairs** as it runs very poorly and doesn't actually make your code any simpler. There is no simple replacement for ``pairs``. If you regularly need to loop through your data try to structure it in a way that doesn't need ``pairs``.
### Forbidden technique
The main reason you would need to use ``pairs`` is if your table needs to act as a lookup table in addition to being looped through. You can actually loop through it a lot cheaper by separating the two purposes into different tables:
```lua
local lookup = {}
local keys = table.newarray() -- see next section

for i = 1, #keys do
    local v = lookup[keys[i]]
    ...
end
```
When you set a key in the lookup, add it to the keys list if it isn't already there, or remove it if it's being set to nil. I don't recommend doing this except when performance is critical, because it adds code complexity as well as making insertion/removal more expensive (as you have to do it to two tables now). TODO: does a custom class using metatables to implement this automatically perform comparably?
## Use array tables!!
A useful feature Kahlua has that standard Lua does not have is array tables. Arrays are tables that only accept integer keys (starting from 1). Internally they are implemented using an array rather than a hash map, which results in them being around twice as fast to access/assign to. **Trying to access or assign invalid keys will throw an error**.

To create an array, use ``table.newarray()`` in place of ``{}``. You can pass any number of arguments as default members of the array like the regular table constructor. If you pass a table (or another array) and nothing else, the array will copy that table instead.

**Arrays are still tables**. You can pass one to any function that expects one, and ``type(myArray)`` will return ``"table"``.

These are often a free optimisation as treating tables like arrays is already highly recommended - you can change one line of code and your array-style table is now twice as fast.

There are a couple caveats to their use. Arrays cannot be saved, so **you can't put them in mod data or send them with network commands**. Additionally, many functions that expect tables will attempt to access or assign keys that are invalid for arrays, which will cause errors and possibly unintended behaviour. If you only need to do this very irregularly, you can clone the array into a regular table before these operations. However if you need to do this regularly the cost of cloning it is too large and you should just use a regular table.

[Return](../README.md)
