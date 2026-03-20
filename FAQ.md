# FAQ / Index

### Q: Will this work with X or Y pack ?
If this pack was made for Optifine: **No**.
VCIT uses its own resource format, and has a fundamentally different approach to CITs.


### Q: What are the main differences with optifine ?
Optifine works by defining variants on a case-by-case basis. VCIT works by defining a general rule for an item, and letting the mod figure out its variants and their associated models.

VCIT works best for packs that have many variants for a single item.
Its goal is to handle as many variants as possible using a few files as possible, and yield better performances in the most extreme cases.

Variants-CIT v5 introduced a [`predicates`]() module type that works more similarly to optifine. However, its usage should only be considered as a band-aid solution for scenarios that regular modules cannot handles.

### Q: How do I port a pack ?
Follow the [introductory tutorial](./Getting%20Started). Then pick a [module type](./Module-Types) that best matches your use case.


### Q: Changing an item's look based on its name.
Follow the [introductory tutorial](./Getting%20Started), which does precisely that.

### Q: Matching prefixes, suffixes, and other patterns in names

Instead of using a `custom_name` module, use a [`component_data`](./Module-Types#component_data) module with a [Regex Transform](./Item-Properties#transform-regex).

#### Example:
```json
{
	"type": "component_data",
	"items": "...",
	"modelPrefix": "...",
	"parameters": {
		"componentType": "custom_name",
		"transform": [
			{
				"regex": "prefix_(.+)_suffix",
				"substitution": "$1"
			},
			"sanitize_path"
		]
	}
}
```

You can use [Regex 101](https://regex101.com/) to test your regexes and substitution strings. Make sure to select the "Java" flavor and the "Substitution" function in the left panel.

Remember that unlike optifine, VCIT does not work by matching values, but by *transforming* raw data into a variant ID. Similarly, the regex here does not simply validate the name's format, it extracts a variant from the name, using _capturing groups_ and a _substitution_ string

See also: [Regex-related Issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3A%22regex%22)


### Q: Using X or Y component as the variant.
Check whether there is a [purpose-made module](./Module-Types#purpose-made-modules) for your use case. Otherwise, use a [`component_data`](./Module-Types#component_data) module.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#property-item_component)


### Q: Using multiple components or multiple pathes.
Use a [`component_format`](./Module-Types#module-component_format) module instead of `component_data`.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#property-item_component)


### Q: Add arbitrary checks or requirements, without necessarily using them as variants.
For checking the existence or validity of some invariant data on an item, use the [`precondition`](./Module-Configuration#field-precondition) field of a module.

For defining variants that don't neatly follow any rule, use a [`predicates`](./Module-Types) module type.

Example: Enchantment-based variant, but only on swords that have a specific custom-data:
```jsonc
{
	// Automatic enchantment-based variants
	"type": "enchantment_vector",
	"modelPrefix": "enchanted_end_sword/",
	"assetGen": "item_model/handheld",

	// Invariant requirements
	"items": "diamond_sword",
	"precondition": {
		"custom_data.hp_item_type": "END_SWORD"
	}
}
```


### Q: Using threshold values in `component_data`/`component_format`.
**Not Supported.**

Some other modules do support thresholds: [`component_threshold`](./Module-Types#module-component_threshold), [`item_count`](./Module-Types#module-item_count), [`enchantment_vector`](./Module-Types#module-enchantment_vector-stored_enchantment_vector), etc.

If all else fail, you can also try using a [`predicates`](./Module-Types#module-predicates) module type.


### Q: Variant based on text formatting.
**Not supported.** Under consideration, but no ETA. Please open an issue describing your use cases, to help shape a potential upcoming feature.


### Q: Changing the look of equipped armor.
In short, add those fields to your modules:
```jsonc
{
	"hook": "equippable",
	"assetGen": "equipment/humanoid"
}
```

See [Equipable Modules](https://github.com/Estecka/mc-Variants-CIT/wiki/Equipped%20Armor) for detailled instructions.


### Q: Changing the look of the trident's projectile.
**Not supported.** Projectiles are rendered as entities, not as item stacks.


### Q: Weapons/tools in the player's hand are held incorrectly, or do lack have animations.
Set the "`assetGen`" field of your module to the appropriate preset. Common tools use `item_model/handheld`. Items with animations have more specific presets such as `item_model/bow` or `item_model/trident`.
See "[Built-in Asset Generator Presets](./Asset-Generation#built-in-asset-generator-presets)" for the complete list of posssible values.

For items with no animation, you can also use the simpler "[`modelParent`](./Module-Configuration#field-modelparent)" field instead of "`assetGen`".

If no preset matches your item, you will need to provide your own item states. You can provide them directly as plain variant assets (see: [`modelPrefix`](./Module-Configuration#field-modelprefix)), or you can write a [custom asset generator](./Asset-Generation#custom-asset-generators) to write most of those assets for you. Either way, this requires a good understanding of how vanilla resource packs.

Item states are a vanilla feature. If you have trouble getting item states to work, you can also try seeking help from other minecraft communities; they probably have more resources and will be more reactive than I.
See also: [Item Model Definition](https://minecraft.wiki/w/Items_model_definition), [Item-states related issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3Aitem-states)


### Q: Animated textures.
Just use the vanilla format for your textures. This means putting all your frames in a single png, with a corresponding [`.png.mcmeta`](https://minecraft.wiki/w/Resource_pack#Texture_animation) file.

If you have trouble getting vanilla animations to work, also try seeking help from other Minecraft communities; they probably have more resources and will be more reactive than I.


### Q: Missing models/textures (magenta checkerboard), but the models work in optifine.
__**Do not use Optifine, Cit-Resewn, or Blockbench as indicators that your models are valid.**__

Your json models probably rely on **features that are exclusive to optifine,** those models are not compatible with vanilla minecraft and need to be modified. Unlike Optifine, Variants-CIT does not change the inner-working of models; all it does is swap one model for another.

Known issues are:
- The resource location of models or textures [uses a bad formatting](https://github.com/Estecka/mc-Variants-CIT/issues/42#issuecomment-2746369948)
- The assets are stored in an [invalid directory](https://github.com/Estecka/mc-Variants-CIT/issues/40#issuecomment-2711744555)


### Q: Missing models when using ModernFix. (MC 1.21.4 and later)
Either disable ModernFix's [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ) feature, or [bake Variants-CIT's generated assets](./Asset-Generation#baking-generated-assets).
