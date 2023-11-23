# Script Function Signatures
In many of Zomboid's script formats there are entries for Lua functions that should be called under a certain circumstance. Similarly to events, these have specific parameters passed to them. These functions must be accessible through the global namespace under the name specified in the script. In most cases these can (and should) be within a table (exceptions noted in their sections).

**Note: while it is unavoidable when using script functions, you should avoid using the global namespace in your mods as much as possible. See [Modules](Modules.md) for more information on this.**

Contents |  |
--- | ---
[Items](#items) | [OnCreate](#oncreate)<br>[OnCooked](#oncooked)<br>[OnEat](#oneat)<br>[AcceptItemFunction](#acceptitemfunction)
[Recipes](#recipes) | [OnCanPerform](#oncanperform)<br>[OnTest](#ontest)<br>[OnCreate](#oncreate-1)<br>[OnGiveXP](#ongivexp)
[Vehicle Parts](#vehicle-parts) | [init](#init)<br>[create](#create)<br>[checkEngine](#checkengine)<br>[checkOperate](#checkoperate)<br>[update](#update)<br>[use](#use)
[Install/Uninstall tables](#installuninstall-tables) | [test](#test)<br>[complete](#complete)


# Items
Note that when the type of a parameter is InventoryItem, it could actually be any subclass - whichever one that particular item is (generally defined by its Type property).
## OnCreate
Called when the item is first created, before it is placed into its container. This is mainly useful for initialising random values for items.

This function is called on the server only.

Name | Type | Description
--- | --- | ---
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item being created
## OnCooked
Called when the item is cooked. This will not fire if the item has a ReplaceOnCooked as the item instance is destroyed and replaced by the new item

This function is called on the server only.

**Due to an error in the game's code, OnCooked functions cannot be within tables - they must be directly within the global namespace.**
Name | Type | Description
--- | --- | ---
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item being cooked
## OnEat
Called when a player eats the item. This can be used to apply special effects upon eating a specific food item.

This function is called on the client eating the item only.

Name | Type | Description
--- | --- | ---
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item being eaten
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character eating it
amount | float | The amount of the item that was eaten (where 1 is the entire item, 0.5 is 50%, etc)

## AcceptItemFunction
Called when deciding whether an item is allowed within the container which has this property. The container's OnlyAcceptCategory takes precedence over this if one is specified.

If this function returns true (or any value that evaulates to true), the item is allowed in the container. If it returns false or nil, the item is not allowed.

This function is called on the client using the container only.

Name | Type | Description
--- | --- | ---
container | [ItemContainer](https://projectzomboid.com/modding/zombie/inventory/ItemContainer.html) | The container the item is being added to
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item being added to the container

# Recipes
All recipe functions are called on the client doing the crafting only.
## OnCanPerform
Called when checking whether a character is able to perform the recipe. This is mostly useful when something beyond the recipe itself is required to craft. If you want to check the items involved, [OnTest](#ontest) is better.

Name | Type | Description
--- | --- | ---
recipe | [Recipe](https://projectzomboid.com/modding/zombie/scripting/objects/Recipe.html) | The recipe being checked
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character being checked 
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item the player right clicked on to see this recipe. Will be null if the player is using the recipe menu instead
## OnTest
Called when checking if an item is suitable for use in the recipe. This can be used to exclude items that match certain criteria, e.g. disallowing items that are not at full durability.

If this function returns true (or any value that evaluates to true), the item is accepted into the recipe. If it returns false or nil, the item is rejected. The return value does not necessarily mean the recipe can or cannot be crafted, it just decides whether this specific item will be considered for it.

Name | Type | Description
--- | --- | ---
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item being checked
result | [Recipe.Result](https://projectzomboid.com/modding/zombie/scripting/objects/Recipe.Result.html) | Describes some details of the result item of the recipe

## OnCreate
Called after successfully crafting the recipe. A common use of this is to grant multiple or random items from a recipe, or implement a recipe that does something other than creating an item.

Name | Type | Description
--- | --- | ---
sources | [ArrayList](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html)\<[InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html)\> | The items used to create the recipe
result | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item crafted by the recipe. This will be passed even if the recipe has the RemoveResultItem property set
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character who crafted the recipe
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item used in the crafting action - this is either the item that was selected in the inventory to start the recipe, or the first item in sources
isPrimaryHandItem | boolean | True if item is the item in the player's main hand
isSecondaryHandItem | boolean | True if item is the item in the player's secondary hand
## OnGiveXP
Called after crafting the recipe. These functions are intended to grant XP, but there is no functional difference between this and OnCreate other than the parameters.

Name | Type | Description
--- | --- | ---
recipe | [Recipe](https://projectzomboid.com/modding/zombie/scripting/objects/Recipe.html) | The recipe that was crafted
sources | [ArrayList](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html)\<[InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html)\> | The items used to craft the recipe
result | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item crafted by the recipe
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character who crafted the recipe
# Vehicle Parts
Vehicle part functions are set using a table in the part script rather than as properties.
```
lua {
    test = MyModGlobals.Test.MyVehicle,
}
```
## init
Called when the part is initialised. This is every time the part loads in. This function is also called when the part or vehicle's repair() method is called (not regular in-game repairs), as this is essentially a reset of the vehicle.

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being initialised
## create
Called when the part is created. This only happens when the vehicle is spawned/loaded for the first time ever, or if a new part has been added to the vehicle's script. This is useful for setting initial/random properties.

This function is called before the part's random condition is assigned, so it cannot be read from, but if it sets one it will not be overriden, so it is safe to set the condition at this stage.

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being created
## checkEngine
Called when checking if the engine is working. This isn't just called for the engine, but every part that has a checkEngine function. If any of them don't return true, the engine is considered not working. This is checked every tick that the engine is running, and the engine will immediately shut off and the ui will reflect it not working.

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being checked
## checkOperate
Called on every part every tick while a player is in the driver seat and able to drive.

If this function does not return true, the player will not be able to control the vehicle.

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being checked
## update
Called when the part updates. This function is throttled so that it is never called more often than once every half an in-game minute. (averaging 1.25 seconds on default settings) If you are attempting to apply an effect over time, you can use the minutes parameter as a [delta](GameTime.md).

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being updated
minutes | float | The number of minutes since the last update.

## use
If a part has a use function, when a player interacts with the vehicle while in the part's area, it will call this function instead of attempting to enter the vehicle.

Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being used
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character using the part.

## Install/Uninstall tables
Part install/uninstall tables can also have lua functions attached. Both use the same functions.
### test
Called when testing if this part can be installed/uninstalled. If it returns true, then the part can be installed/uninstalled.

This function is called only on the client attempting to install the part.
Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being used
character | [IsoGameCharacter](https://projectzomboid.com/modding/zombie/characters/IsoGameCharacter.html) | The character trying to install/uninstall the part.
### complete
Called when a part is finished being installed/uninstalled.

This function is called only on the server.
Name | Type | Description
--- | --- | ---
vehicle | [BaseVehicle](https://projectzomboid.com/modding/zombie/vehicles/BaseVehicle.html) | The vehicle the part belongs to
part | [VehiclePart](https://projectzomboid.com/modding/zombie/vehicles/VehiclePart.html) | The part being used
item | [InventoryItem](https://projectzomboid.com/modding/zombie/inventory/InventoryItem.html) | The item that was removed. Only passed if the part was uninstalled

[Return](../README.md)
