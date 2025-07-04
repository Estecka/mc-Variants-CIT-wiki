# Pack inter-compatibility
This page is about best practices for maximising the compatibility between multiple overlapping texture packs.

Suppose you have two texture packs. One pack provides CITs for vanilla enchanted book, the other provides CITs for modded enchantments. These packs can easily be made compatible, but there are a few pitfalls to avoid:
- **One pack overwrite the other's module.** If both pack provide differently configured modules at the same path, the top-most module will overwrite the bottom one. If the two modules are configured identically, this will not be an issue. You can move your modules to your own namespace.
- **One pack's module provides a fallback model.** Even if both modules are loaded, the one with a fallback model is unlikely to ever hand over control to the other. You can ensure the module without a fallback has a higher priority than the other one.

In all cases, even if the two packs' modules diverge to preserve the full functionality of both, you can still offer basic compatibility by ensuring your CITs use the same naming scheme (`modelPrefix` and other separators) as the other pack. This will allow one pack's CITs to be detected and used by the other pack's module.

# Recommended Configurations

The list of modules below covers some common use cases. Should you decide to include a module at one of the pathes specified below, it should be configured _exactly_ as provided here. If you need to make any modification, move the module to your own namespace to avoid conflicts with other packs.

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
For level-invariant CITs:  
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

For level-based CITs:  
`/assets/minecraft/variants-cit/item/enchanted_book_levels.json`
```json
{
	"type": "stored_enchantment",
	"priority": 1,
	"items": ["enchanted_book"],
	"modelPrefix": "item/enchanted_book_levels/",
	"modelParent": "item/generated",
	"parameters": {
		"levelSeparator": "."
	},
	"special": {
		"multi": "item/multi_enchanted_book"
	}
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
