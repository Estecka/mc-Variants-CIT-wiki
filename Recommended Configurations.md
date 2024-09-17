# Recommended Configurations

This page is about best practices for maximising the compatibility between multiple overlapping texture packs.

For example, you could have one pack that provides CITs for vanilla enchanted book, and a another pack that provides CITs for modded enchantments.
If both pack provide differently configured modules at the same path, the top-most pack's module will completely overwrite the bottom one. If the modules define different `modelPrefix`es, than one packs CITs will be ignored. 
So long as none of the modules provide a `fallback` model, the safest way to make those packs work together is for them to provide two separate modules at two separate pathes. (The path of the module has no bearing on its functionality, so long as its `items` is explicitely defined.) 
However if, in addition to having different prefixes, the modules provide any `fallback` model, then one module's fallback may still take priority over the other's CITs. In this case, the only way to make the packs work together is to make sure they are set up to use the exact same module.

The list of modules below covers some of the most common use cases. Should you decide to include a module at one of the pathes below, it should be configured _exactly_ as provided here. Doing so will ensure your pack remains compatible with other packs that use the same module.

## Axolotl Buckets
`/assets/minecraft/variants-cit/item/axolotl_bucket.json`
```json
{
	"type": "axolotl_variant",
	"modelPrefix": "item/axolotl_bucket_",
	"modelParent": "item/generated"
}
```
This is the same naming scheme used by the mod *More Axolotl Variants*

## Enchanted Books
`/assets/minecraft/variants-cit/item/enchanted_book.json`
```json
{
	"type": "stored_enchantment",
	"modelPrefix": "item/enchanted_book/",
	"modelParent": "item/generated",
	"special": {
		"multi": "item/multi_enchanted_book"
	}
}
```

Models in the variant directory should only contain level-invariant models.
Storing level-based models in the variant directory will cause each level to be detected as its own enchantment. This is unlikely to cause bugs, but is less optimised. Level-based models should be stored in a separate directory. I recommend `enchanted_book_levels`.

In order to assign each model to the corresponding level, use the `level` predicate to define overrides inside the level-invariant models.  
Example: `/assets/minecraft/models/item/enchanted_book/unbreaking.json`
```json
{
	"parent": "item/enchanted_book_levels/unbreaking_1",
	"overrides": [
		{
			"predicate": { "level": 2 },
			"model": "item/enchanted_book_levels/unbreaking_2"
		},
		{
			"predicate": { "level": 3 },
			"model": "item/enchanted_book_levels/unbreaking_3"
		}
	]
}
```

## Goat Horns
`/assets/minecraft/variants-cit/item/goat_horn.json`
```json
{
	"type": "instrument",
	"modelPrefix": "item/goat_horn/",
	"modelParent": "item/generated"
}
```
These models should have a tooting override, just like the vanilla model.
Additional models for tooting should be stored in a separate directory: `item/tooting_goat_horn/`.
You can set your models' parents to the corresponding vanilla models, this will save you the need to redefine the "display" section.

## Music Discs
Relevant for datapacks that include custom songs.

All vanilla discs being different items, their are two ways to go about it. If you're confident your custom songs will only end up on a specific disc, you can create a module specific to it:

`/assets/minecraft/variants-cit/item/music_disc_<name>.json`
```json
{
	"type": "jukebox_playable",
	"modelPrefix": "item/music_disc_",
	"modelParent": "item/generated"
}
```

Otherwise you may create a single module encompassing all vanilla discs:

`/assets/minecraft/variants-cit/item/music_disc.json`
```json
{
	"type": "jukebox_playable",
	"items": [
		"music_disc_5",
		"music_disc_13",
		"music_disc_11",
		"music_disc_blocks",
		"music_disc_cat",
		"music_disc_chirp",
		"music_disc_creator",
		"music_disc_creator_music_box",
		"music_disc_far",
		"music_disc_mall",
		"music_disc_mellohi",
		"music_disc_precipice",
		"music_disc_otherside",
		"music_disc_pigstep",
		"music_disc_relic",
		"music_disc_stal",
		"music_disc_strad",
		"music_disc_wait",
		"music_disc_ward"
	],
	"modelPrefix": "item/music_disc_",
	"modelParent": "item/generated"
}
```
Should you need to include modded disc types, creating a separate module for this mod is more compatible than adding those discs to the "vanilla" module.

Both modules use the same naming scheme as the vanilla disc textures, and will be compatible with each other.

## Painting Variants

`/assets/invarpaint/variants-cit/item/painting.json`
```json
{
	"type": "painting_variant",
	"modelPrefix": "item/painting/",
	"modelParent": "item/generated",
	"fallback": "invarpaint:item/filled_painting",
	"special": {
		"invalid": "invarpaint:item/missing_painting"
	}
}
```
This is the exact same module that is used by the mod *Invariable Paintings*. It uses the `invarpaint` namespace for some CITs, and for the module itself.

## Potions
Being different items, splash potions and lingering potions require a different modules:

`/assets/minecraft/variants-cit/item/potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/potion/",
	"modelParent": "item/generated"
}
```

`/assets/minecraft/variants-cit/item/splash_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/splash_potion/",
	"modelParent": "item/generated"
}
```

`/assets/minecraft/variants-cit/item/lingering_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/lingering_potion/",
	"modelParent": "item/generated"
}

```
