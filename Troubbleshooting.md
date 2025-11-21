# Troubleshooting
## Debug tools

There are a few client-side commands that you can use to check the behaviours of your modules.
```
/variants-cit <context> <module id> [summary|dump|walkthrough]
```
`context` will be either "`item_model`" (by default) or "`equippable`" depending on what has been defined in your module.

> [!TIP]
>
> You can use `F3 + D` to clear the in-game chat. For commands that give a lot of feedback, this can make it easier to see where things start.

### Command: `summary`
On most module, `summary` will simply give you the amout of variants that this module manages.

On `enchant_vector` modules, this will also give the list of unique enchantments that are present on your models. This can be a quick way to check if any of those models has a mispelled name.

### Command: `dump`
This will give you the complete list of models that this module manages, and their associated variant ID. This can also be used to check whether the module collected models or textures it was not supposed to, which may be the result of an insufficiently specific model prefix.

On `enchantment_vector`, this will give you the set of enchantments each model is associated to, instead of their variant ID.

### Command: `walkthrough`
This command will forcibly run the given module on the item in your main hand, and give you information on what the module is trying to do and why it may have failed.
The details vary from one module type to another, but at minimum, this will tell you what variant ID was found if any, does this variant have a model, and what that model might be.

![walkthrough](./walkthrough_command.png)

## Common Issues
### Issue: Models are left unchanged
Put the item you want to change in your main hand, and run the command:
```
/variants-cit <context> <module id> walkthrough
```
Depending on what the result tells you:

#### > "No such module exists."
If your module's ID is not listed by the autocomplete, it means there is an error in the module's json file. Open the game's log, and look for an error that describes the issue. This error will appear during resource-reload.

If no error appears at all, your module is probably stored in the wrong folder.

#### > "This module would normally not apply to items of type xxxx."
Your item's type was not listed in the module's target [`items`](./Module-Configuration#items).

#### > "The item has a valid variant, but no associated model exists."
The command will also give you the variant ID of your item, and what your model/texture should be named. Make sure that the file name matches exactly what is given.

Note that if you want to provide only textures but no baked model, you must also define a [model parent](Module-Configuration#modelParent) in the module, otherwise textures will be ignored.

Instead of `walkthrough`, you can also use the `dump` commands to check whether your existing models were collected by the module, and if so, what variant ID they were associated with.

#### > "No variant could be computed for this item."
The item does not provide the data this module expected to find.

If you are using a purpose-made module, double-check that the item provides the required component, and that you are using the correct module for the job.

If you are using `component_data` or `component_format`, look for additional messages below.

#### > "Raw Data: Missing or Invalid"
The item does not provide the data this module expected to find.

First, check that the module's `nbtPath` matches the location of the data. You can check the item's nbt structure by running this command with the item still in your main hand :
```
/data get entity @s SelectedItem.components
```

Secondly, check that the module's [`expect`](./Item-Properties#field-expect) is set to the type of data you are looking for. Primitive types will be handled correctly by default, but some texts can only be decoded using `rich_text` or `rich_text_array`.  
In doubt set it to `auto`, and see if this fixes it. The type `auto`, can be used for quick troubleshooting, but its behaviour may change as new types are added, so it should not be used in released packs.

#### > "Transformed: null"
The [`transform`](./Item-Properties#transforms) associated with this data failed to produce an output. In particular, if you are using a regex, it may mean that the regex does not match the input.


### Issue: Missing models or textures
(A.k.a. the pink and black checkerboard.)

If you use ModernFix, disable [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ).

If you provided only textures, it means that you provided an invalid `modelParent`

If you are providing custom json models, it means those models contains errors.  
Even if those models originate from a working optifine pack, they may still be invalid. Optifine allows for pack structures and asset formats that are illegal in vanilla minecraft. If you are trying to port an optifine-formatted pack, you may still need to make modifications to its models, and move some files to directories where Minecraft will actually load them.

Possible issues caused by Optifine are:
- The resource location of models or textures [uses a bad formatting](https://github.com/Estecka/mc-Variants-CIT/issues/42#issuecomment-2746369948)
- The assets are stored in an [invalid directory](https://github.com/Estecka/mc-Variants-CIT/issues/40#issuecomment-2711744555)

If possible, try testing your models and item states in pure-vanilla minecraft, by using the `item_model` component instead of a VCIT module.

### Issue: Items aren't held correctly
Check that you are using the correct model parent. `item/generated` for regular items, `item/handheld` for regular tools and weapons.

### Issue: Item animations are missing
Models for weapons with complexes animations like bows, shields and tridents cannot be generated automatically from textures. For those, you'll must provide custom item states and baked models, using the vanilla assets as an example.

If possible, try testing your models and item states in pure-vanilla minecraft, by using the `item_model` component instead of a VCIT module.
