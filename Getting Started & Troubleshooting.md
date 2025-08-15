# Getting Started
## The basics
Variants-CIT's format is fundamentally different from Optifine. In order to get your bearings, this will walk you through the process of changing a diamond sword based on a name given in an anvil. This assumes you already have some basic understanding of [namespaced identifiers](https://minecraft.wiki/w/Identifier), and how to [create and organize resource packs](https://minecraft.wiki/w/Tutorial:Creating_a_resource_pack).

### 1. Create the CIT module

The module is the object that decides when to change the item's texture, and what texture to use.
A single module can manage multiple variants, and does not care about the exact list of variants. You'll usually only need a single module per item, regardless of how many textures you have.

Create a JSON file anywhere in `assets/<namespace>/variants-cit/item/<path>.json`. The `<namespace>` and `<path>` of the file doesn't affect its functionality. Your name should nonetheless be unique, since other packs can overwrite each other's modules.

The file will have this content:
```json
{
	"type": "custom_name",
	"items": "minecraft:diamond_sword",
	"modelParent": "minecraft:item/handheld",
	"modelPrefix": "item/custom_diamond_sword/"
}
```

- `type` can be anything from [this list](Module-Types). Simply changing the module's type is enough to change its behaviour; the rest of the pack's structure will remain the same in almost all cases.


- `modelParent` will usually be `item/handheld` for tools and weapons, or `item/generated` for almost all other items.

- `modelPrefix` can be whatever your want, but keep note of what you choose; this is the folder where we'll put our textures later on.
The trailing slash '`/`' at the end of the prefix is important !

  Starting MC 1.21.4, the leading `item/` in the model prefix is implied, but if you want your pack to be compatible with MC 1.21.3 and earlier, you should always specify it.

See [here](Module-Configuration#modelPrefix) for an advanced quide on how to configure modules.


### 2. Add textures to the pack

Suppose we want to use a custom texture for a sword named **"Épée de l'End"**.

We need to figure out the so-called **"variant id"** of this item. Finding a variant id is a common step for all module types, but the way it is calculated varies from one type to another. Again, refers to the [type list](Module-Types) to learn how a specific module finds its variants.


In the case of `custom_name`, the module creates the variant by sanitizing the item's name, removing any illegal characters:
"Épée de l'End" becomes "`epee_de_lend`", then is given the default namespace: "`minecraft:epee_de_lend`". This is our variant id. The variant id is then combined with the previously chosen model prefix, which gives us a model id of: `minecraft:item/custom_diamond_sword/epee_de_lend`.

In your pack, place your texture at:  
`/assets/minecraft/textures/item/custom_diamond_sword/epee_de_lend.png`  
and that's it! The mod will know to use this texture for this name, you texture pack is now functional.

If you want to support more custom names, you don't need to make any change to the module; just add more textures to the pack, and name them appropriately.

### 3. Using custom models

Up to this point, this will work if you just want a flat texture for your sword. But what if you wanted to use a flashier model ?

It's more of the same, just provide a similarly named [json model](https://minecraft.wiki/w/Model), and the mod will automatically pick it up:  
`/assets/minecraft/models/item/custom_diamond_sword/epee_de_lend.json`

Similarly, if you want to leverage the functionalities of MC 1.21.4's new [item states](https://minecraft.wiki/w/Items_model_definition), just provide an appropriately named file:  
`/assets/minecraft/items/custom_diamond_sword/epee_de_lend.json`

Ultimately, all the mod does is override the value of the `item_model` component of the item, meaning everything that can be done with those can be done with Variants-CIT. Having a good understanding of how this component works will help you a lot.

Variant ID                         | `<namespace>:<path>`
---------------------------------- | :-------------------
Equivalent `item_model` component  | `<namespace>:<modelPrefix><path>`
Matching item state                | `/assets/<namespace>/items/<modelPrefix><path>.json`
Matching baked model               | `/assets/<namespace>/models/item/<modelPrefix><path>.json`
Matching texture                   | `/assets/<namespace>/textures/item/<modelPrefix><path>.png`

## Using custom data as variants
The `custom_name` module type we used above requires little configuration, but is designed around a specific use case. There are other [purpose-made modules](./Module-Types#purpose-made-modules) for other common use cases, but if you need a variant that is stored into an unusual location, the two modules types that will interest you are [`component_data`](./Module-Types#component_data) and [`component_format`](./Module-Types#component_format); those let you pick data from anywhere in a chosen component's NBT representation.

Relevant documentation: [Item Properties / Transforms](./Item-Properties)

### Locating the data
Suppose you want to create a module for the effect of suspicious stew:
First, you need to figure out where the variant is stored in the item. Go in-game, put a bowl of supicious stew in your main hand, and use this command:  
```
/data get entity @s SelectedItem.components
```

 This should print something like this into the chat:

![{"minecraft:suspicious_stew_effects": [{duration:7, id:"minecraft:saturation"}]}](./nbt_path_command.jpg)

Here we learn that: (1) The effect id is in a component called `suspicious_stew_effect`. (2) The effect is stored under a key "id", which itself is stored at the first position of an array.

Fill this into the `componentType` and ['`nbtPath`'](./Item-Properties#nbtpath) parameters of component_data, and your module will use the effect id as the variant id:
```json
{
	"type": "component_data",
	"items": "minecraft:suspicious_stew",
	"modelPrefix": "item/stew/",
	"parameters": {
		"componentType": "suspicious_stew_effects",
		"nbtPath": "[0].id"
	}
}
```

> [!TIP]
> 
> Specific components and data types may require extra `"expect"` and `"transform"` parameters.
> For now, you can just set them to auto:
> ```json
> {
> 	"componentType": "...",
> 	"nbtPath": "...",
> 	"expect": "auto",
> 	"transform": "sanitize_auto"
> }
> ```
> I wouldn't recommend using these values in released packs, but you can use them as training wheels.
> It's easier to get something working with them, but they may also reduce performances, or treat some invalid components as valid.

### Combining multiple pieces of data
The module `component_data` can only work with a single piece of data, `component_format` is its equivalent for working with multiple pieces of data.

Here's a module that constructs a variant id from both a firework's explosion pattern, and its flight duration:
```json
{
	"type": "component_format",
	"items": "firework_rocket",
	"modelPrefix": "item/rocket/",
	"parameters": {
		"format": "${pattern}_${duration}",
		"variables": {
			"pattern":  { "componentType": "fireworks", "nbtPath": ".explosions[0].shape" },
			"duration": { "componentType": "fireworks", "nbtPath": ".flight_duration" }
		}
	}
}
```

The objects inside "variables" are formatted the same way as for the `component_data` module. The variant id is built using the provided `format`, where `${variables}` are substitued with their corresponding value.

### Sanitizing the data

By default, variants are expected to be stored as plain identifiers. If the data you're looking for contains some illegal characters, you'll need extra parameters in order to convert it into a valid variant ID.

Hypixel uses upper-case identifiers, minecraft identifiers only allows lower-cases. You can convert those ids to lower-case like this:
```json
{
	"componentType": "custom_data",
	"nbtPath": ".id",
	"transform": "lowercase"
}
```

For more complex texts similar to custom names, you'll want two things:
- `"expect":"rich_text"` will let the module know the text may not be stored as a plain string, but instead as an NBT text. **This will strip all formatting from the text,** there is currently no reliable way to use formatting in the variant id.
- `"transform":"sanitize"` wil perform a much more aggressive sanitization, identically to the `custom_name` module mentioned in the beginning.

```json
{
	"componentType": "lore",
	"nbtPath": "[0]",
	"expect": "rich_text",
	"transform": "sanitize"
}
```

If you want to only use a portion of the text, or require it to match a specific format, you can use regular expressions to transform the data.
For example, this will only match custom names that start with a given prefix, and remove that prefix from the variant-id:
```json
{
	"componentType": "custom_name",
	"nbtPath": "",
	"expect": "rich_text",
	"transform": [
		"sanitize",
		{ "regex":"someprefix_(.+)", "substitution":"$1"}
	]
}
```
Here, the regex is applied _after_ sanitization, but you can change the order of the transforms, or add new ones as you see fit.

I recommend you use [regex101.com](https://regex101.com/) to test your regular expressions and substitution strings.

# Troubleshooting

## Models are left unchanged

### Check that the module and its models are properly loaded.
Hit F3+T (reload resource packs), then look at Minecraft's console log. Look for a line that says _**"Found X variants for CIT module <module_id>"**_.
(The module id is `<namespace>:<path>` for the file `assets/<namespace>/variants-cit/item/<path>.json`)

If you cannot find the line with your module's id, it means the module itself could not be loaded. There may be an error in the format of your json, in which case additional errors will be shown in the log. Otherwise, the module may not be located in the correct directory.

If the line exists, but the number of variants is wrong, it means there is an issue with the file structure of your models/textures; make sure their path match the [model prefix](Module-Configuration#modelPrefix) defined in your module. If you are providing textures without any model, make sure your module has a [model parent](Module-Configuration#modelParent).

### Check the variant ID and the data structure of the item.
If both the module and models are loaded, then it means you model names don't match the item's variant ID, or that the module could not find the variant ID.

In game, put an item in your main-hand, and use the the command `/data get entity @s SelectedItem.components` in order to quickly check that the components on the item match the one expected by the module.
For NBT-based modules, this will also show the structure of the component, so you can check your nbtPath.

Modules that perform some heavy transformation on the data (such as `custom_name` or `component_data`) can take `"debug":true` as parameter, which will print the final variant IDs of any item it encounters into the console log.
For other module types, double check [how the variant ID is computed](./Module-Types).



## Missing models or textures
(A.k.a. the pink and black checkerboard.)

If you use ModernFix, try disabling [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ).

Otherwise, double check that the baked models, model parents, or item states are correct.

Optifine allows for pack structures and asset formats that are illegal in vanilla minecraft.
If you are trying to port an optifine-formatted pack, even if the models are already provided to you, you may still need to make modifications to them, and move some files around to directories where Minecraft will actually load them.

If possible, try testing your models and item states in vanilla minecraft, by using the `item_model` component instead of a CIT module.

## Items aren't held correctly, missing animations
Check that you are using the correct model parent. `item/generated` for regular items, `item/handheld` for regular tools and weapons.

Models for weapons with complexes animations like bows, shields and tridents cannot be generated automatically from textures. For those, you'll need to provide custom item states and baked models, using the vanilla assets as an example.

If possible, try testing your models and item states in vanilla minecraft, by using the `item_model` component instead of a CIT module.
