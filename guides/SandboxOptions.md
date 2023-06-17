# File Format and Setup
The sandbox option file must be named ``sandbox-options.txt`` and must be in the ``media/`` folder of your mod.

The first line of your file must be ``VERSION = 1,`` . This tells the game what version of the sandbox option format your file is, so that if the developers ever change the format the game will be able to handle it automatically. Do not change this number.

# Option Blocks
Each option is defined in an option block. A generic option block looks like this:
```
option MyMod.Name {
    type = KEEP READING,

}
```
Note that this is stripped down to the common parts for demonstration purposes, and isn't actually a valid sandbox option.

The most important part is the ``option MyMod.Name``. This defines the name of our option: The format is \[Table].\[OptionName], with the table usually being named after the mod.
You'll be able to access the option in Lua using the name you set using ``SandboxVars.[Table].[OptionName]``. For example, an option defined as ``option MyMod.TestOption`` can be accessed using ``SandboxVars.MyMod.TestOption``.
You don't have to include a table name if you don't want to, however I do not recommend ever doing this as it makes conflicts with other mods much more likely. You can't add nested tables (e.g. ``MyMod.Submodule.TestOption`` will not work).

The name you put here won't be displayed to players: you'll need translation strings to set the name of your option, explained in the [Translation](#translation) section.

The type declares what datatype the option represents. The available types are [boolean](#boolean), [integer](#integer), [double](#double), [string](#string), [enum](#enum).
**Booleans** are yes/no tickboxes, **integers** are whole numbers, **doubles** are decimal numbers, **strings** expect the player to type something in, and **enums** are lists of options.

# Option Types
## Boolean
```
default = true/false,
```
Booleans are as simple as it gets. The option will be a tickbox. You can set the tickbox to be ticked by default (true) or unticked (false).
## Integer
```
min = -5,
max = 10,
default = 5,
```
Integers are whole numbers (no decimal point). If you need decimal numbers, use a double instead. You can set the minimum, maximum and default values for the number.
## Double
```
min = -9.5,
max = 12.34,
default = 1,
```
Doubles are decimal numbers. If you need only whole numbers, use an integer instead. You can set the minimum, maximum and default values for the number.
## String
```
default = MyString,
```
Strings are text. You only need to set a default value for these - if you want the default to be empty, write ``default = ,`` (even though it looks wrong!)
## Enum
```
numValues = 3,
default = 1,
```
Enums are a list of options to pick one from. You need to set how many options there are, and which one is the default. Internally this is basically the same as an integer, but displayed differently to users.
You'll need translation strings to name the options, explained in the next section.
# Translation
```
page = MyPage,
translation = MyMod_OptionName,
```
All options use translation strings the same way.
## Enum values
```
valueTranslation = MyMod_OptionName_Values,
```
# Example of a full Sandbox Option file

# Common Issues

# Cheat Sheet
