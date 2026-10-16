# Changing Equipped Armor
## Prerequisites
This tutorial assumes you already have some familiarity with the mod, or have at least followed the introductory tutorial:
- [Getting Started](./Getting-Started)

This page also touches on some concepts of asset generation, but those are only required for handling rare edge cases:
- [Understanding Asset-Generation](./Understanding-AssetGen)

### Index
- [Equippables](#equippables)
	- [In short](#in-short)
	- [Non-humanoid armors](#non-humanoid-armors)
	- [Equipment Assets](#equipment-assets)
	- [Asset-Generation quirks](#asset-generation-quirks)
- [Trim Patterns](#trim-patterns)
	- [In short](#trim-patterns)
	- [Hiding Trims](#hiding-trims)
	- [Trim Assets](#trim-pattern-assets)
	- [Coloured Trim Atlas](#coloured-trim-atlases) (MC 26.2 and prior)

## Equippables
### In short:
Write your module like you would for changing the inventory texture, but set the `hook` of your module to `"equippable"`, and its `assetGen` to one of the `"equipment/xxxx"` assetGen presets:

```json
{
	"items": [
		"diamond_helmet",
		"diamond_chestplate",
		"diamond_leggings",
		"diamond_boots"
	],
	"hook": "equippable",
	"assetGen": "equipment/humanoid",

	"...": "..."
}
```

**Only the textures of equipments can be changed in this way, not their 3D model.**


For humanoid armors set `assetGen` to `"equipment/humanoid"`, and place your textures at:  
`/assets/<namespace>/textures/entity/equipment/humanoid/<model-id>.png`  
`/assets/<namespace>/textures/entity/equipment/humanoid_leggings/<model-id>.png`  
(The same module will work for all four body-parts of an armor set.)

For elytras, use `"equipment/elytra` and put your textures in the `wings` layer:  
`/assets/<namespace>/textures/entity/equipment/wings/<model-id>.png`  

### Non-humanoid armors
This hook works for all items that make use of the `equippable` component: Pet armor, Llama Carpets, Saddles, etc. Look for the matching [assetGen preset](./Asset-Generation#built-in-asset-generator-presets), and put your texures in the folder for the approriate layer(s).

- **[Equipment Layer Types](https://minecraft.wiki/w/Equipment#Equipment_Layer_Types)**: All items that can theoretically be supported by Variants-CIT.
- **[Equipment History](https://minecraft.wiki/w/Equipment#History)**: What version of Minecraft each layer type was introduced in.

### Equipment Assets
In the previous tutorial, the hook of your module was implicitely set to `item_model`, which caused it to override the `item_model` component of the target item.  
For `equippable`, everything works fundamentally the same, excepts the module overrides the `assetId` field inside the '`equippable`' component. The module will collect [Equipment Models](https://minecraft.wiki/w/Equipment) instead of item models, and asset-generation will look for textures in `textures/entity/equipment/`.

Model-ID                       | `<namespace>:<path>`
------------------------------ | :-------------------
`asset_id` inside `equippable` | `<namespace>:<path>`
Matching equipment model       | `/assets/<namespace>/equipment/<path>.json`
Matching texture               | `/assets/<namespace>/textures/entity/equipment/<layer>/<path>.png`

### Asset-Generation quirks
Each equipment model can have multiple textures, corresponding to the `<layer>` in the texture path above. The same module works for all body parts of an armor set, because each model it generates can includes the textures for all layers of an entity.

The existence of a single layer's texture is enough to trigger the creation of a whole model, but asset generators can only create JSON files with a fixed structure. The generated models will always have *all* the layers defined in their template, even if some of the corresponding textures were not provided.

Conversely, if multiple modules provide different layers for the same model but are using different templates, the layers from the two models will *NOT* be combined; only one of the generators will prevail at random, in a classic case of asset-gen conflict.
In this case, you will need either a custom generator that provides all the layers in a single template, or to change the modules so they use models with different IDs.

# Trim patterns
To change the look of trim patterns, use the `trim_pattern` hook, the `trim_pattern/humanoid` assetGen preset, and place your textures at:  
`/assets/<namespace>/textures/trims/entity/humanoid/<model-id>.png`  
`/assets/<namespace>/textures/trims/entity/humanoid_legging/<model-id>.png` 


```jsonc
{
	"items": [
		"diamond_helmet",
		"diamond_chestplate",
		"diamond_leggings",
		"diamond_boots"
	],
	"hook": "trim_pattern",
	"assetGen": "trim_pattern/humanoid",
	"...": "..."
}
```

Similarly to equipables, each trim module can handle all the bodyparts of an armor set. 
Only humanoid armors are supported.

> [!IMPORTANT]
>
> **On MC 26.2 and prior, you must manually add your trim textures to the armor trim atlas, else they will show as missing textures.** ([See further below](#coloured-trim-atlases)).

### Hiding trims
Modules can't *remove* trims per-se, but they can replace them with an empty texture. Variants-CIT already includes such a model under the ID `variants-cit:null`.
Simply make your module return this model.

With `predicates`:
```jsonc
"parameters": {
	"predicates": [
		{
			"modelId": "variants-cit:null",
			"precondition": { "..." }
		}
	]
}
```

With any Variant-ID based module:
```json
"modelList": {
	"some-variant-id": "variants-cit:null"
}
```

For this particular model no `assetGen` option is required, and you do not need to edit the sprite atlas. All the required assets are already baked in the mod.

### Trim Pattern Assets
This hook overrides the value of the `asset_id` in the trim pattern referenced by the `trim` component.


Model-ID:                        | `<namespace>:<path>`
-------------------------------- | :-------------------
`asset_id` inside trim patterns: | `<namespace>:<model-id>`
Matching trim model:             | `/assets/<namespace>/variants-cit/trim_pattern/<model-id>.json`
Matching texture:                | `/assets/<namespace>/textures/trims/entity/<layer>/<model-id>.png`

Textures are the only assets you need to care about. The "trim models" are but a technicality, used by the mod to represent the textures of different layers under a single file. Although trim patterns have a vanilla resource type representing them, those resources are *server-side* and VCIT cannot interact with them as easily.

There exist only a single `assetGen` preset for this hook, and you will not need any other.

### Coloured trim atlases
> [!NOTE]
>
> This section is relevant only to **MC 26.2 and prior**.
>
> Since MC 26.3, no action is required from you for trims to work.

Trim textures are unique in that they need to have coloured variations for every material.
VCIT does not automate this, and Minecraft only partially automates the creation of coloured trim textures. For those to be created, you must manually add your textures to the `armor_trim` atlas.

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
