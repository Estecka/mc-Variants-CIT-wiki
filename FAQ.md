# FAQ / Index

### Q: Will this work with X or Y pack ?
If this pack was made for Optifine: **No**.
VCIT uses its own resource format, and has a fundamentally different approach to CITs.


### Q: What are the main differences with optifine ?
Optifine works by defining variants on a case-by-case basis. Variants-CIT works by defining a "module" for an item, which manages multiple variants at once.

VCIT works best for packs that have many variants for a single item.
Its goal is to handle as many variants as possible using as few files as possible, and yield better performances in the most extreme cases.

Historically, the main design philosophy for modules was to have a single generic rule that lets the mod automatically figure out an item's variants, and their associated models. Today, there's also a [`predicates`](./Module-Types#module-predicates) module type that works more similarly to optifine, on case-by-case basis.

### Q: How do I port a pack ?
Follow the [introductory tutorial](./Getting%20Started). Then pick a [module type](./Module-Types) that best matches your use case.


### Q: How do I change an item's look based on its name ?
Follow the [introductory tutorial](./Getting%20Started), which does precisely that.

### Q: How do I match prefixes, suffixes, and other patterns in names ?

Instead of using the `custom_name` module showcased in the tutorial, use a [`component_data`](./Module-Types#component_data) module with a [Regex Transform](./Transforms#transform-regex).

#### Example:
```json
{
	"items": "...",
	"modelPrefix": "...",

	"type": "component_data",
	"parameters": {
		"componentType": "custom_name",
		"transform": [
			{
				"function": "sanitize"
			},
			{
				"regex": "prefix_(.*)_suffix",
				"substitution": "$1"
			}
		]
	}
}
```

You can use [Regex 101](https://regex101.com/) to test your regexes and substitution strings. Make sure to select the "Java" flavor and the "Substitution" function in the left panel.

Remember that unlike optifine, VCIT does not work by matching values, but by *transforming* raw data into a variant ID. Similarly, the regex here does not simply validate the name's format, it extracts a variant from the name, using _capturing groups_ and a _substitution_ string

See also: [Regex-related Issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3A%22regex%22)


### Q: How do I use this or that component as the variant ID ?
Check whether there is a [purpose-made module](./Module-Types) for your use case. Otherwise, use a [`component_data`](./Module-Types#component_data) module.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#property-item_component)


### Q: How can I use multiple components to build a variant ID ?
Use a [`component_format`](./Module-Types#module-component_format) module instead of `component_data`.

See also: [Item properties](./Item-Properties) and ['`item_component`' property](./Item-Properties#property-item_component)


### Q: Add arbitrary checks or requirements.
For checking some invariant data on an item, use the [`precondition`](./Module-Configuration#field-precondition) field of a module.

For checking data that directly affects the item's variant, you may use a [`predicates`](./Module-Types#module-predicates) module type (if no other module matches your use-case).

Example:
```jsonc
{
	"modelPrefix": "enchanted_end_sword/",
	"assetGen": "item_model/handheld",

	// Invariant requirements do not affect which model is used, but are still required to apply a model at all.
	"items": "diamond_sword",
	"precondition": {
		"custom_data.server_item_type": "END_SWORD"
	},

	// Case-by-case variants
	"type": "predicates",
	"parameters": {
		"predicates": [
			{
				"variantId": "ascended",
				"precondition": { "custom_data.ascended": 1 }
			}
			{
				"variantId": "reforged",
				"precondition": { "custom_data.reforged": 1 }
			}
		]
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


### Q: Weapons/tools in the player's hand are held incorrectly, or do not have animations.
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
