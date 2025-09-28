# FAQ

### Q: Will this work with X or Y pack ?
If this pack was made for Optifine: **No**.
VCIT uses its own resource format, and has a fundamentally different approach to CITs.


### Q: What are the main differences with optifine ?
Optifine works by defining variants on a case-by-case basis. VCIT works by defining a general rule for an item, and letting the mod figure out its variants and their associated models.

VCIT works best for packs that have many variants for a single item.
Its goal is to handle as many variants as possible using a few files as possible, and yield better performances in the most extreme cases.


### Q: How do I port a pack ?
Follow the [introductory tutorial](./Getting%20Started%20&%20Troubleshooting). 

For some specific uses cases, you will also want to learn how to use the vanilla ['`item_model`' component](https://minecraft.wiki/w/Items_model_definition).


### Q: Changing an item's look based on its name.
Follow the [introductory tutorial](./Getting%20Started%20&%20Troubleshooting), which does precisely that.

### Q: Matching prefixes, suffixes, and other patterns in names

Instead of using a `custom_name` module, use a [`component_data`](./Module-Types.md#component_data) module with a [Regex Transform](./Item-Properties.md#regex-transform).

#### Example:
```json
{
	"type": "component_data",
	"items": "...",
	"modelPrefix": "...",
	"parameters": {
		"componentType": "custom_name",
		"expect": "rich_text",
		"transform": [
			"sanitize_path",
			{
				"regex": "prefix_(.+)_suffix",
				"substitution": "$1"
			}
		]
	}
}
```

You can use [Regex 101](https://regex101.com/) to test your regexes and substitution strings. Make sure to select the "Java 8" flavor and the "Substitution" function in the left panel.

Remember that unlike optifine, VCIT does not work by matching values, but by *transforming* raw data into a variant ID. Similarly, the regex here does not simply validate the name's format, it extracts a variant from the name, using _capturing groups_ and a _substitution_ string

See also: [Regex-related Issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3A%22regex%22)


### Q: Using X or Y component as the variant.
Check whether their is a [purpose-made module](./Module-Types#purpose-made-modules) for your use case. Otherwise, use a [`component_data`](./Module-Types#component_data) module.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#item_component)


### Q: Using multiple components or multiple pathes.
Use a [`component_format`](./Module-Types#component_format) module instead of `component_data`.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#item_component)


### Q: Add arbitrary checks or requirements for what items are affected by a module.
Technically not a feature, however, this should be possible to achieve using the [regex transform](./Item-Properties#regex-transform) in a `component_format` module.

Defining variables that are not used in the format is currently allowed, but is still considered a fringe case. Please leave a comment [here](https://github.com/Estecka/mc-Variants-CIT/issues/57) describing your use case, to help shape a potential upcoming feature.


### Q: Using threshold values in `component_data`/`component_format`.
**Not Supported.** Under consideration, but no ETA, and will definitely require deep restructuring of those modules. Some [purpose-made modules](./Module-Types#purpose-made-modules) do support thresholds, and adding new ones is easier for the time being. Please open an issue describing your use case.


### Q: Variant based on text formatting.
**Not supported.** Under consideration, but no ETA. Please open an issue describing your use cases, to help shape a potential upcoming feature.


### Q: Changing the look of equipped armor.
**Experimental.**
Modules for this are practically identical to regular modules, but need to be provided with [`equipments`](https://minecraft.wiki/w/Equipment) as CITs, (instead of Item States and Baked Models for regular modules).

See: [Equipable Modules](https://github.com/Estecka/mc-Variants-CIT/wiki/Equipped%20Armor)


### Q: Changing the look of the trident's projectile.
**Not supported.** Projectiles are rendered as entities, not as item stacks.


### Q: Weapons/tools in the player's hand are held incorrectly.
Use `item/handheld` as the [`modelParent`](./Module-Configuration#modelparent) in your module, or as the `parent` in your [baked models](https://minecraft.wiki/w/Model#Item_models).


###	Q: Shield blocking, Bow pulling, Trident in hand, Throwing trident, Fishing Rod cast, Goat Horn tooting and other item-specific actions.
These items use [item states](https://minecraft.wiki/w/Items_model_definition) to control their animations, which VCIT cannot generate automatically. You must provide these item states in your pack; one for each variant, like for baked models and textures. Just copy the vanilla item states for those items, and swap out the model names with your custom ones.

Item states are a vanilla feature, if you have trouble getting item states to work, also try seeking help from other minecraft communities; they probably have more resources will be more reactive than I.

See also: [`modelPrefix`](./Module-Configuration#modelprefix), [Item-states related issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3A%22items%20states%22)


### Q: Animated textures.
Just use the vanilla format for your textures. This means putting all your frames in a single png, with a corresponding [`.png.mcmeta`](https://minecraft.wiki/w/Resource_pack#Texture_animation) file.

If you have trouble getting vanilla animations to work, also try seeking help from other Minecraft communities; they probably have more resources and will be more reactive than I.


### Q: Missing models/textures (magenta checkerboard), but the models work in optifine.
__**DO NOT TEST YOUR MODELS WITH OPTIFINE OR CIT-RESEWN INSTALLED.**__

Your baked models or item states probably rely on features that are exclusive to optifine, those files are not compatible with vanilla minecraft and need to be modified.

Known issues are:
- The resource location of models or textures [uses a bad formatting](https://github.com/Estecka/mc-Variants-CIT/issues/42#issuecomment-2746369948)
- The assets are stored in an [invalid directory](https://github.com/Estecka/mc-Variants-CIT/issues/40#issuecomment-2711744555)


### Q: Missing models when using ModernFix. (MC 1.21.4 and later)
Disable ModernFix's [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ) feature.
