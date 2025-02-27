# Getting Started

This will walk you through the process of changing a diamond sword based on a name given in an anvil. This assumes you already have some basic understanding of [namespaced identifiers](https://minecraft.wiki/w/Identifier), and how to [create and organize resource packs](https://minecraft.wiki/w/Tutorial:Creating_a_resource_pack).

### 1. Create the CIT module

Create a JSON file anywhere in `assets/<namespace>/variants-cit/item/<path>.json`. The `<namespace>` and `<path>` of the file doesn't affect its functionality.

The file will have this content:
```json
{
	"type": "custom_name",
	"items": ["minecraft:diamond_sword"],
	"modelParent": "minecraft:item/handheld",
	"modelPrefix": "item/custom_diamond_sword/"
}
```

`type` can be anything from [this list](Module-Types). Simply changing the module's type is enough to change its behaviour; the rest of the pack's structure will remain the same in almost all cases.


`modelParent` will usually be `item/handheld` for tools and weapons, or `item/generated` for almost all other items.

`modelPrefix` can be whatever your want, but keep note of what you choose.
 The trailing slash '`/`' at the end of the prefix is important !

Starting MC 1.21.4, the leading `item/` in the model prefix is implied, but if you want your pack to be cross-compatible with MC 1.21.3 and earlier, you should always specify it.

See [here](Module-Configuration#modelPrefix) for an advanced description of the possible configurations.


### 2. Add textures to the pack

Suppose we want to use a custom texture for a sword named **"Épée de l'End"**.

We need to figure out the so-called "variant id" of this item. Finding a variant id is a common step for all module types, but the way it is calculated varies from one type to another. Again, refers to the [type list](Module-Types) to learn how a specific module finds its variants.


In the case of `custom_name`, the module creates the variant by sanitizing the item's name, removing any illegal characters:
"Épée de l'End" becomes "`epee_de_lend`", then is given the default namespace: "`minecraft:epee_de_lend`". This is our variant id. The variant id is then combined with the previously chosen model prefix, which gives us a model id of: `minecraft:item/custom_diamond_sword/epee_de_lend`.

In your pack, place your texture at:  
`/assets/minecraft/textures/item/custom_diamond_sword/epee_de_lend.png`  
and that's it! The mod will know to use this texture for this name, you texture pack is now functional.

### 3. Using custom models

Up to this point, this will work if you just want a flat texture for your sword. But what if you wanted to use a flashier model ?

It's more of the same, just provide a similarly named [json model](https://minecraft.wiki/w/Model), and the mod will automatically pick it up:  
`/assets/minecraft/models/item/custom_diamond_sword/epee_de_lend.json`

Similarly, if you want to leverage the functionalities of MC 1.21.4's new [item states](https://minecraft.wiki/w/Items_model_definition), just provide an appropriately named file:  
`/assets/minecraft/items/custom_diamond_sword/epee_de_lend.json`


# Troubleshooting

## Models are unchanged

### Check that the module and its models are properly loaded.
Hit F3+T (reload resource packs), then look at Minecraft's console log. Look for a line that says _**"Found X variants for CIT module <module_id>"**_.
(The module id is `<namespace>:<path>` for the file `assets/<namespace>/variants-cit/item/<path>.json`)

If you cannot find the line with your module's id, it means the module itself could not be loaded. There may be an error in the format of your json, in which case additional errors will be shown in the log. Otherwise, the module may not be located in the correct directory.

If the line exists, but the number of variants is wrong, it means there is an issue with the file structure of your models/textures; make sure their path match the [model prefix](Module-Configuration#modelPrefix) defined in your module. If you are providing textures without any model, make sure your module has a [model parent](Module-Configuration#modelParent).

### Check the variant ID and the data structure of the item.
If both the module and models are loaded, then it means you model names don't match the item's variant ID, or that the module could not find the variant ID.

In game, put an item in your main-hand, and use use the the command `/data get entity @s SelectedItem.components` in order to quickly check that the components on the item match the one expected by the module.
For NBT-based modules, how those components are structured, this will also show the structure of the component, so you can check your nbtPath.

Modules that perform some heavy transformation on the data (such as `custom_name` or `component_data`) can take `"debug":true` as parameter, which will print the final variant IDs of any item it encounters into the console log.
For other module types, double check [how the variant ID is computed](./Module-Types).



## Missing models
If you use ModernFix, try disabling [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ).

Otherwise, double check that the baked models, model parents, or item states are correct.

## Items aren't held correctly
Check that you are using the correct model parent. `item/generated` for regular items, `item/handheld` for regular tools and weapons.

Models for weapons with complexes animations like bows and tridents cannot be generated automatically from textures. For those, you'll need to provide custom item states and baked models, using the vanilla assets as an example.
