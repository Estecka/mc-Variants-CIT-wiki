# Troubleshooting

This page is targeted at resource-pack makers rather than end-users.

### In short:
1. Use `moduletree crawl` to check which module applies to an item
1. Use `module walkthrough` to check why a given module did or did not change the item's model.
1. Use `module variant-id` or `module model-id` to check why a specific variant was or was not included in the module.

### Index
#### Debug Commands
- [`module summary`](#command-summary)
- [`module dump`](#command-dump)
- [`module walkthrough`](#command-walkthrough)
- [`module variant-id`](#command-variant-id-model-id)
- [`module model-id`](#command-variant-id-model-id)
- [`moduletree crawl`](#command-crawl)
#### Common issues
- [Models are left unchanged, or are changed to the wrong model.](#issue-models-are-left-unchanged)
- [Missing models or texture](#issue-missing-models-or-textures)
- [Items aren't held correctly, or model animations are missing](#issue-items-arent-held-correctly-or-model-animations-are-missing)


# Debug commands
The general syntax for commands is:
```
/variants-cit module <hook> <module id> ...
/variants-cit moduletree <hook> ...
```
- `<hook>` can be "`item_model`", "`equippable`" or "`trim_pattern`" depending on what has been defined in your module. the defualt is "item_model".
- `<module id>` depends on the file path to your module's JSON file. It is relative to `variants-cit/modules/`

> [!TIP]
>
> You can use `F3 + D` to clear the in-game chat. Some commands give a lot of feedback, and this makes it easier to find where they start.


### Command: `summary`
```
/variants-cit module <hook> <module id> summary
```

On most module, `summary` will simply give you the amout of variants that this module manages, and the components it relies on.

On `enchant_vector` modules, this will also give the list of unique enchantments that are present on your models. This can be a quick way to check if any of those models has a mispelled name.


### Command: `dump`
```
/variants-cit module <hook> <module id> dump
```
This gives the complete list of models that was collected by this module, and their associated variant ID.


### Command: `variant-id`, `model-id`
```
/variants-cit module <hook> <module id> variant-id <identifier>
/variants-cit module <hook> <module id> model-id <identifier>
```

Gives information about what the given id represents for this module, such as what variant or model is it associated with, or why it was or was not collected by the module.
The module `enchantment_vector` will also give the exact list of enchantments it parsed from the tested variant ID.

The autocomplete will suggest IDs that are relevant to the module, but any ID can be tested here.


### Command: `walkthrough`
```
/variants-cit module <hook> <module id> walkthrough [[self|nearest_item|nearest_player]]
```
This command will forcibly run the given module on the item in your main hand, and give you information on what the module is trying to do and why it may have failed.
The details vary from one module type to another, but at minimum, this will tell you what was the raw value of the item's relevant components, what variant ID was found if any, does this variant have a model, and what that model might be.

![walkthrough](./walkthrough_command.png)


### Command: `crawl`
```
/variants-cit moduletree <hook> crawl [[self|nearest_item|nearest_player]]
```
This command tests all modules that can potentially apply to the item in your main-hand. It reports whether the modules failed or succeeded, and which one actually took control of the item's model.


# Common Issues
### Issue: Models are left unchanged
Use the following commands to debug your item. The commands will target the item in your main-hand by default. You can target an item on the ground by adding `nearest_item` at the end of the command.

If you have multiple modules affecting that item, run `moduletree crawl` to check which module is taking control of the item right now.
```
/variants-cit moduletree <hook> crawl [[nearest_item]]
```

If you have identified a module that incorrectly failed or succeeded, use `module walkthrough` to test this module in particular:
```
/variants-cit module <hook> <module id> walkthrough [[nearest_item]]
```
Depending on what the result tells you:

#### > "No such module exists"
Your module is probably stored in the wrong folder, or its name contains inalid characters.

#### > "Module failed to load"
Your JSON contains errors. The command will also print the error message that happened when loading the module.

#### > "This module would normally not apply to items of type xxxx."
Your item's type was not listed in the module's target [`items`](./Module-Configuration#field-items).

#### > "Precondition failed"
Your module's [precondition](./Precondition%20Cheat-Sheet) did not match the item.
You can try inserting [log](./Transforms#transform-log) transforms into your preconditions, and run `walkthrough` again to try and find the specific transform that failed.

#### > "The item has a valid variant, but no associated model exists."
The command will also give you the variant ID of your item, and what your model/texture should be named. Make sure that the file name matches exactly what is given.

You can use `module variant-id` and `module model-id` to get more informations on why your model failed to be recognized.

If you provide only textures but no JSON model, you must add an [assetGen](./Module-Configuration#field-assetgen) option to your module.

#### > "No variant could be computed for this item."
The item does not provide the data this module expected to find, or a transform failed to process that data.

If you are using a purpose-made module, double-check that the item provides the required component, and that you are using the correct module for the job.

If you are using `predicates`, you can insert [log](./Transforms#transform-log) transforms into your preconditions, and run `walkthrough` again to try and find the specific transform that failed.

If you are using `component_data` or `component_format`, look for additional messages below.

#### > "Raw Data: Missing or Invalid"
The item does not provide the data this module expected to find.

Check that the `nbtPath` matches the location of the data. You can check the item's nbt structure by running this command with the item still in your main hand :
```
/data get entity @s SelectedItem.components
```

#### > "Transformed: null"
The transform chain failed to process the data.

In particular, if you are using a regex, it may mean that the regex did not match the input. Use [Regex 101](https://regex101.com/) to test your regex against the input, and check that the substitution result is what you expect it to be.

More gerally, you can insert [log](./Transforms#transform-log) at various points into your transform to try and figure out where it has failed exactly.


### Issue: Missing models or textures
(A.k.a. the pink and black checkerboard.)

If you are changing trim pattern textures, make sure you added those textures to the [armor trim sprite atlas](./Equipped-Armor#coloured-trim-atlases).

If you use ModernFix, either disable [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ), or [bake Variants-CIT's assets](./Asset-Generation#baking-generated-assets).

If you are using `modelParent` instead of `asssetGen`, make sure the parent model exists and is valid.  
If you are using `assetGen` with a custom asset-generator, make sure it generates all the required assets

If you are directly providing your own json models, it means those models contains errors.  
**Even if those models originate from a working Optifine pack, they may still be invalid.** Optifine allows for pack structures and asset formats that are illegal in vanilla minecraft. You may need to make modifications to its models, and move some files to different directories where Minecraft will actually load them. 
Check the game's log for relevant errors. Possible issues caused by Optifine-formatted assets are:
- The resource location of models or textures [uses a bad formatting](https://github.com/Estecka/mc-Variants-CIT/issues/42#issuecomment-2746369948)
- The assets are stored in an [invalid directory](https://github.com/Estecka/mc-Variants-CIT/issues/40#issuecomment-2711744555)

If possible, try testing your models and item states in pure-vanilla minecraft, by using the `item_model` component instead of a VCIT module.

### Issue: Items aren't held correctly, or model animations are missing
Check that you are using the correct `assetGen` option:
- Use `item_model/generated` for regular items.
- Use `item_model/handheld` for basic tools and weapons.
- For any other item, check the list of [builtin assetGen presets](./Asset-Generation#built-in-asset-generator-presets) to find the one matching your item.
