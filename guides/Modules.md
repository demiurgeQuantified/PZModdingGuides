# Avoiding the Global Namespace and Modules

## What is the global namespace?
The global namespace is the default, shared environment of all files in Lua. Any variables declared without the ``local`` keyword are added to the global namespace[*](#notes) (they are known as 'globals'). This is as opposed to local variables, which are limited to the scope they're declared in.

## Why is it bad?
There are two major issues with the global namespace:
- It's slower than local variables
- It's prone to name conflicts

The first point is fairly simple: internally, the global namespace is actually a table (you can access it under the global ``_G``), which means that any usage of a global incurs the cost of a table access. As the global namespace in PZ already contains hundreds of variables the speed of this is further impacted.

The second is the fact that only one variable under any given name can exist. If two mods declare global variables with the same name, they'll overwrite each other which can cause many kinds of issues. An example of this is if two mods were to define a function called ``test()``:
```lua
function test()
    print("Mod A is working!")
end
```
```lua
function test()
    print("Mod B is working!")
end
```
Whichever file loads last will overwrite the function from the previous file, so only that one will appear to be working when the function is called. This is very frustrating to debug, as nothing in the 'broken' mod is wrong.

## Solutions
### Just don't use globals
Seriously. Just make everything local. Most variables and functions don't need to be accessed from other files anyway, and local variables can be pre-declared to share them between scopes within the same file. If your variable is only used within one function, or within one file, there is absolutely no benefit to it being global and making it local is essentially a free performance boost and mod conflict safety.[**](#notes)

But what if you do need the same variable/function in multiple files? And what if other mods need to be able to access/override your functions for compatibility?
### Modules
```lua
-- At the start of the file
local MyModule = {}

-- At the very end of the file (nothing after this line will execute)
return MyModule
```
This is all that is required for the module pattern (of course, MyModule can and should be called whatever you want.) Anything you want to be accessible from other files should be placed within this table, and anything else should be global.

```lua
local MyModule = {}

MyModule.myField = 2

-- i prefer these 'backwards' function declarations
MyModule.myFunction = function()
    return 5
end

-- but the 'normal' style still works too
function MyModule.anotherFunction()
    return 10
end

return MyModule
```

To use this module in another file, you just need to ``require`` the file. This returns the module table.
```lua
local MyModule = require "path/to/my/module"
```
The path to a file should start after the ``shared/client/server`` part of the directory. For example, to require ``shared/MyMod/MyModule.lua``, you would write ``require "MyMod/MyModule"``.[***](#notes) You can include the ``.lua`` extension if you wish.

```lua
local MyModule = require "MyMod/MyModule"

print(MyModule.myField) -- > 2
print(MyModule.myFunction()) -- > 5
```

A hidden benefit of modules is that they ensure a load order. When you require a file it checks if the file has been loaded yet, and loads it if it hasn't. This prevents a lot of inconsistencies and means that the burden of load order isn't placed on the end users who wouldn't be able to tell which mods should load first anyway.

You cannot require files from a folder that hasn't loaded yet. Files load in the order shared -> client -> server: requiring a file from a folder later in this order always fails. You can cheat this by delaying the require until after that folder would have loaded using events, but I would recommend structuring your code to not need it.

This does lead to one complication though: if File A requires File B, and File B requires File A, both need to load before each other, which isn't possible. This is called a **circular require** and will cause the require to fail. However the solution to this is fairly simple: there is not really any situation where you should need a circular require to begin with. If you need code from both files in each other, you can just split the shared code into a third module. In a way, this is almost an upside: it forces you to stop your code from getting too tangled.

## Alternative solutions (and their problems)
A pattern popular in many mods and even PZ's vanilla lua is to keep all variables/functions under a single globally accessible table, similar to a module:
```lua
MyMod = {}
-- Mods that use multiple files may use this variant instead, which lets all of the files use the same global without overwriting each other
MyMod = MyMod or {}
```
However this is more of an antipattern as it still declares one global when it could declare none just by writing a module instead.

## When you have to use globals
There are two main situations where you do have to use globals:
- [Script Functions](ScriptFunctions.md) as they can only search the global namespace
- When you are overwriting a global from another mod/vanilla (otherwise you're just overriding a local copy of it and not the original)

#### Notes
*A variable declared without the ``local`` keyword is not actually always added to the global namespace - it is added to the environment of the declaring file or function, which is usually the global namespace, but it can be configured to be a different table instead. However this is very rarely used so it is not important to understand.

**There is a limit of 200 locals per scope, including upvalues. This is far more than enough, but if it somehow becomes an issue, you can use ``do end`` blocks to separate parts of a scope into separate smaller scopes.

***Because of this, if you have files with the same path in more than one of the folders, you can't disambiguate between them in require calls. I don't know what happens in this situation, just don't do it.

[Return](../README.md)
