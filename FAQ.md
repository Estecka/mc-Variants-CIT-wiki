# FAQ

### Q: Will this work with X or Y pack ?
If this pack was made for Optifine: **No**.  
VCIT uses its own resource format, which is not based on anything Optifine-like.


### Q: What are the main differences with optifine ?
Optifine works by defining variants on a case-by-case basis. Variants-CIT works by defining a "module" for an item, which manages multiple variants at once.

Certain module types can automatically associate models to the corresponding item variant, based on the model's name. This makes it possible to add several dozens of variants to an item, by writing only a handful of lines of code in a single file.

### Q: How do I port or create a pack ?
Start by follow the [introductory tutorial](./Getting-Started).

VCIT uses a completely different format from Optifine, so there is no quick and easy way to port a pack.


### Q: How do I change an item's look based on its name ?
Start by following the [introductory tutorial](./Getting-Started), which has examples of that.

Do consider that there are three flavours of name to choose from:
- `custom_name` is the name given in an anvil.
- `item_name` is the intrinsic name of the item, regardless of custom name. (Sometimes incorrectly refered to as "command-given name")
- `display_name` is whichever name is shown to the player.

In doubt, use `display_name`.


### Q: How do I check text formatting ?
**Limited support only.**
Use [`nbt_path`](./Transforms#transform-nbt_path) to navigate through the NBT structure of a rich text. You may also use [`get_snbt`](./Transforms#nbt) to get the String representation of the NBT, and check it against a Regex.

A features using Legacy Formatting Codes is under consideration, but has no ETA.


### Q: How do I check custom data ?
In `predicates` modules, add the path to the data directly after the name of the component:
```json
{
	"items": "...",

	"type": "predicates",
	"parameters": {
		"predicates": [
			{
				"modelId": "excalibur",
				"precondition": {
					"custom_data.id": "EXCALIBUR"
				}
			}
		]
	}
}
```

See also: [Item properties](./Item-Properties), [Precondition](./Precondition%20Cheat-Sheet)


### Q: Can I change the look of equipped armor ?
Yes. In short, add those fields to your modules:
```jsonc
{
	"hook": "equippable",
	"assetGen": "equipment/humanoid"
}
```

See [Equipable Modules](https://github.com/Estecka/mc-Variants-CIT/wiki/Equipped%20Armor) for detailled instructions.


### Q: Can I change the look of the trident's projectile ?
**Not supported.** Projectiles are rendered as entities, not as item stacks.


### Q: Can I add arbitrary conditions to modules other than `predicates` ?
Every module can have a `precondition` block at its root, which works similarly to the conditions used in `predicates` modules :
```jsonc
{
	"item": "diamond_sword",
	"precondition": {
		"custom_data.server_item_type": "END_SWORD"
	},

	"modelPrefix": "enchanted_end_sword/",
	"assetGen": "item_model/handheld",

	"type": "enchantment"
}
```

See also: [Precondition Cheat-Sheet](./Precondition%20Cheat-Sheet#cheat-sheet)


### Q: Can I use threshold values in `component_data`/`component_format` ?
**Not Supported.**
Module types that do support thresholds are: [`component_threshold`](./Module-Types#module-component_threshold), [`durability`](./Module-Types#module-durability), [`item_count`](./Module-Types#module-item_count), [`enchantment`](./Module-Types#module-enchantment-stored_enchantment), [`enchantment_vector`](./Module-Types#module-enchantment_vector-stored_enchantment_vector)


### Q: Weapons/tools in the player's hand are held incorrectly, or do not have animations.
Set the "`assetGen`" field of your module to the appropriate preset. Common tools use `item_model/handheld`. Items with animations have more specific presets such as `item_model/bow` or `item_model/trident`.
See "[Built-in Asset Generator Presets](./Asset-Generation#built-in-asset-generator-presets)" for the complete list of posssible values.

For items with no animation, you can also use the simpler "[`modelParent`](./Module-Configuration#field-modelparent)" field instead of "`assetGen`".

If no preset matches your item, you will need to either provide your own JSON models, or write a [custom asset generator](./Asset-Generation#custom-asset-generators).

See also: [Understanding Asset-Generation](./Understanding-AssetGen), [Item Model Definition](https://minecraft.wiki/w/Items_model_definition), [Item-states related issues](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3Aitem-states)


### Q: How do I make animated textures ?
Just use the vanilla format for your textures. This means putting all your frames in a single png, with a corresponding [`.png.mcmeta`](https://minecraft.wiki/w/Resource_pack#Texture_animation) file.

If you have trouble getting vanilla animations to work, also try seeking help from other Minecraft communities; they probably have more resources and will be more reactive than I.


### Q: Missing models/textures (magenta checkerboard), but the models work in optifine.
__**Do not use Optifine, Cit-Resewn, or Blockbench as indicators that your models are valid.**__

Those models may still rely on **features that are exclusive to optifine,** and need to be modified so as to be compatible with Vanilla Minecraft.

Known issues in optifine-formatted models are:
- The resource location of models or textures [uses a bad formatting](https://github.com/Estecka/mc-Variants-CIT/issues/42#issuecomment-2746369948)
- The assets are stored in an [invalid directory](https://github.com/Estecka/mc-Variants-CIT/issues/40#issuecomment-2711744555)


### Q: Missing models when using ModernFix. (MC 1.21.4 and later)
Either disable ModernFix's [Dynamic Resources](https://github.com/embeddedt/ModernFix/wiki/Dynamic-Resources-FAQ) feature, or [bake Variants-CIT's generated assets](./Asset-Generation#baking-generated-assets).
