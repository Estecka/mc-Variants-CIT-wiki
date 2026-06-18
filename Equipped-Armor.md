# Equippable Modules

Changing the look of equipped armor works similarly to changing inventory textures; this page here will only cover the differences. If you are not aready familiar with Variants-CIT, make sure to follow the [introductory turtorial](./Getting%20Started) first.

The fundamental difference is that equippable modules will override the `assetId` of the ['`equippable`'](https://minecraft.wiki/w/Data_component_format#equippable) component, instead of the `item_model` component previously.

Modules are able to change the look of equipment on any mob that makes use of the `equippable` component, not just humanoids. (See the [History](https://minecraft.wiki/w/Equipment#History) table on the equipment assets wiki.)

Only the textures of equipments can be changed that way, not their 3D model.

## In short:
Set the [`hook`](./Module-Configuration#field-hook) of your module to `"equippable"`. For humanoid armors, set its [`assetGen`](./Module-Configuration#field-assetgen) to `"equipment/humanoid"`.
```json
{
	"hook": "equippable",
	"assetGen": "equipment/humanoid",
	"...": "..."
}
```

It's technically possible for a module to pull double-duty, and change both the equipped and inventory look of an item. However, because of the discrepancy in how `item_model` assets and `equippable` assets are structured, creating two separate modules will be more convenient in most cases.


## Asset Types
Instead of collecting Item States, Baked Models and Textures, equippable modules will collect ['`equipment`' assets](https://minecraft.wiki/w/Equipment).

Variant ID                                          | `<namespace>:<path>`
--------------------------------------------------- | :-------------------
Equivalent `asset_id` in the `equippable` component | `<namespace>:<modelPrefix><path>`
Matching equipment model                            | `/assets/<namespace>/equipment/<modelPrefix><path>.json`
Matching texture                                    | `/assets/<namespace>/textures/entity/equipment/<layer>/<modelPrefix><path>.png`

Equipements models can have multiple layers, meaning you only need a single model (and thus a single module) to handle all the body parts of an armor tier, potentially even across multiple mobs !

## Asset Generation
For equipments, there are only a couple of [generator presets](./Asset-Generation#built-in-asset-generator-presets) which should cover the most common use cases: `equipment/humanoid` (full-body armor) and `equipment/elytra`.

The existence of a single layer's texture is enough to trigger the creation of a model, but generators can only create json's with a fixed structure. As a result, generated models will always have *all* the layers defined by the template, even if some of the corresponding textures were not provided.


# Trim patterns
```jsonc
{
	"hook": "trim_pattern",
	"assetGen": "trim_pattern/humanoid",
	"...": "..."
}
```

To change the look of trim patterns, use the `trim_pattern` hook, and place your textures at:  
`/assets/<namespace>/textures/trims/entity/<layer>/<modelPrefix><path>.png`.

Similarly to equipements, trims need separate textures for the `humanoid` and `humanoid_leggings` layer, and modules can handle all body parts of an armor set.

**Unlike other hooks, you must manually add your textures to the trim atlas, else they'll show as missing textures.** ([See below](#coloured-trim-atlases)).

## Hiding trims
Modules can't remove trims per-se, but can replace them with an empty texture. Variants-CIT already includes such a trim model, which you can add to your module like this:
```jsonc
"modelList": [
	"variants-cit:null"
]
```
or like this:
```jsonc
"modelList": {
	"variants-cit:null": "variants-cit:null"
}
```

Then, just make your module return the variant ID `variants-cit:null`.

(If this is the only model in your module, then no modelPrefix or assetGen options are required.)


## Asset Types
This hook overrides the value of the `asset_id` in the trim pattern of the `trim` component.

There exist only a single `assetGen` preset for this hook, and you will not need any other.

Make sure to register your custom trim textures to the [trim atlas](#coloured-trim-atlases) !


Variant ID:                                    | `<namespace>:<path>`
---------------------------------------------- | :-------------------
Equivalent `asset_id` in the `trim` component: | `<namespace>:<modelPrefix><path>`
Matching trim model:                           | `/assets/<namespace>/variants-cit/trim_pattern/<modelPrefix><path>.json`
Matching texture:                              | `/assets/<namespace>/textures/trims/entity/<layer>/<modelPrefix><path>.png`


Textures are the only assets you really need to care about.   
The "trim models" are a technicality, used by the mod to bring together the different layers.
Although trim patterns have a vanilla resource type representing them, those resources are *server-side*. VCIT cannot interact with them as easily, so it uses a custom assets type instead.

### Coloured trim atlases
Trim textures are unique in that they need to have coloured variations for every material.
VCIT currently does not automate this, and Minecraft only partially automates the creation of coloured trim textures. For those to be created, you must manually add your textures to the armor_trim atlas.

If your trims appear as missing textures, it is likely because you are missing this file in your texture pack:

`/assets/minecraft/atlases/armor_trims.json`
```jsonc
{
	"sources": [
		{
			"type": "paletted_permutations",
			"palette_key": "trims/color_palettes/trim_palette",
			// The list of permutations may vary with your version of minecraft !!
			// E.g: `copper_darker` does not exist until MC 1.21.9, when copper armor was added.
			// In doubt, check the vanilla version of this file and copy the permutations from there.
			"permutations": {
				"amethyst":         "trims/color_palettes/amethyst",
				"copper":           "trims/color_palettes/copper",
				"copper_darker":    "trims/color_palettes/copper_darker",
				"diamond":          "trims/color_palettes/diamond",
				"diamond_darker":   "trims/color_palettes/diamond_darker",
				"emerald":          "trims/color_palettes/emerald",
				"gold":             "trims/color_palettes/gold",
				"gold_darker":      "trims/color_palettes/gold_darker",
				"iron":             "trims/color_palettes/iron",
				"iron_darker":      "trims/color_palettes/iron_darker",
				"lapis":            "trims/color_palettes/lapis",
				"netherite":        "trims/color_palettes/netherite",
				"netherite_darker": "trims/color_palettes/netherite_darker",
				"quartz":           "trims/color_palettes/quartz",
				"redstone":         "trims/color_palettes/redstone",
				"resin":            "trims/color_palettes/resin"
			},
			// List all your trim textures here
			"textures": [
				"minecraft:trims/entity/humanoid/my_trim",
				"minecraft:trims/entity/humanoid_leggings/my_trim"
			]
		}
	]
}
```
