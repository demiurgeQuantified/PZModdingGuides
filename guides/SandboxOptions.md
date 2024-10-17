# File Format and Setup
The sandbox option file must be named ``sandbox-options.txt`` and must be in the ``media/`` folder of your mod.

The first line of your file must be ``VERSION = 1,`` . This tells the game what version of the sandbox option format your file is, so that if the developers ever change the format the game will be able to handle it automatically. Do not change this number.

# Option Blocks
Each option is defined in an option block. A generic option block looks like this:
```
option MyMod.Name {
    type = PLACEHOLDER,

}
```
Note that ``PLACEHOLDER`` is not a valid type. Read on to see valid types.

The most important part is the ``option MyMod.Name``. This defines the name of our option: The format is \[Table].\[OptionName], with the table usually being named after the mod.
You'll be able to access the option in Lua using the name you set using ``SandboxVars.[Table].[OptionName]``. For example, an option defined as ``option MyMod.TestOption`` can be accessed using ``SandboxVars.MyMod.TestOption``.
You don't have to include a table name if you don't want to, however I do not recommend ever doing this as it makes conflicts with other mods much more likely. You can't add nested tables (e.g. ``MyMod.Submodule.TestOption`` will not work).

The name you put here won't be displayed to players: you'll need translation strings to set the name of your option, explained in the [Translation](#translation) section.

The type declares what datatype the option represents. The available types are [boolean](#boolean), [integer](#integer), [double](#double), [string](#string), [enum](#enum).
**Booleans** are yes/no tickboxes, **integers** are whole numbers, **doubles** are decimal numbers, **strings** expect the player to type something in, and **enums** are lists of options.

# Option Types
## Boolean
```
type = boolean,
default = true/false,
```
Booleans are as simple as it gets. The option will be a tickbox. You can set the tickbox to be ticked by default (true) or unticked (false).
## Integer
```
type = integer,
min = -5,
max = 10,
default = 5,
```
Integers are whole numbers (no decimal point). If you need decimal numbers, use a double instead. You can set the minimum, maximum and default values for the number.
## Double
```
type = double,
min = -9.5,
max = 12.34,
default = 1,
```
Doubles are decimal numbers. If you need only whole numbers, use an integer instead. You can set the minimum, maximum and default values for the number.
## String
```
type = string,
default = MyString,
```
Strings are text. You only need to set a default value for these - if you want the default to be empty, write ``default = ,`` (even though it looks wrong!)
## Enum
```
type = enum,
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
All options use translation strings the same way. These strings point to translation strings in ``lua/shared/Translate/EN/Sandbox_EN.txt`` (for other languages, just swap EN for your language code). The file should be formatted as such:
```
Sandbox_EN = {
    Sandbox_MyPage = "My Page Name",
    
    Sandbox_MyMod_OptionName = "My Option Name",
    Sandbox_MyMod_OptionName_tooltip = "Extra details about my option.",
}
```
Note that the strings you put into your option script are prefixed with ``Sandbox_``. Suffixing the option translation with ``_tooltip`` lets you set a tooltip for your option. If you don't want a tooltip, just omit that line.
## Enum values
To name the options in an enum option, you must add to your option script:
```
valueTranslation = MyMod_OptionName_Values,
```
This will correspond to the following translation entries:
```
MyMod_OptionName_Values_option1 = "First Option",
MyMod_OptionName_Values_option2 = "Option Two",
MyMod_OptionName_Values_option3 = "My Third Option :D",
```
and so on, with an option\[number] up to your number of options.
# Example of a working Sandbox Option file
```
VERSION = 1,

option Literacy.XPMultiplier
{
    type = double,
    min = 0.1,
    default = 1.0,
    max = 10.0,
    page = Literacy,
    translation = Literacy_XPMultiplier,
}

option Literacy.IlliteratePenalty
{
    type = enum,
    numValues = 3,
    default = 2,
    page = Literacy,
    translation = Literacy_IlliteratePenalty,
    valueTranslation = Literacy_IlliteratePenalties,
}

option Literacy.WalkWhileReadingLevel
{
    type = integer,
    min = -1,
    default = -1,
    max = 10,
    page = Literacy,
    translation = Literacy_WalkWhileReadingLevel,
}

option Literacy.ReadInTheDark
{
    type = boolean,
    default = true,
    page = Literacy,
    translation = Literacy_ReadInTheDark,
}
```
# Common Issues
- **My sandbox options aren't changing/new ones aren't appearing**

If you've uploaded your mod to the workshop, or you've worked with your mod in multiple directories, it might be reading the wrong file. The game seems to choose which sandbox-options file to load independently of which copy of the mod it's loading, so even if the rest of your mod is loading from the correct directory, **unsubscribe from your mod on the workshop**, or remove it from the directories you aren't using anymore.

---
# Cheat Sheet
Parameter | Types | Type/valid values
--- | --- | ---
type | all | boolean/integer/double/string/enum
page | all | string
translation | all | string
default | all | type of option
min | integer/double | number
max | integer/double | number
numValues | enum | integer
valueTranslation | enum | string

[Return](../README.md)
