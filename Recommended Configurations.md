# Recommended Configurations
Modules being regular resources, it's possible for different packs can completely overwrite each others modules if stored at the same pathes.  it's possible for multiple similar modules to cooexist, but it may be inconvenient to come up with a truly unique name for the modules, and is less optimized.

In order to both reduce the amount of modules per-item, and maximise compatibility between multiple packs, here are some pre-configured modules that cover the most common CIT scenarios. Should your pack include a module at any of the pathes below, it should be configured _exactly_ as provided here, so that other packs may safely rely on it.

## Axolotl Buckets
`/assets/minecraft/variant-cits/item/axolotl_bucket.json`
```json
{
	"type": "axolotl_variant",
	"modelPrefix": "item/axolotl_bucket_"
}
```
This is the same format used by the mod "More Axolotl Variants"

## Enchanted Books
`/assets/minecraft/variant-cits/item/enchanted_book.json`
```json
{
	"type": "stored_enchantment",
	"modelPrefix": "item/enchanted_book/",
	"special": {
		"multi": "item/multi_enchanted_book"
	}
}
```

**Models in the variant directory should only contain level-invariant models.**
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
`/assets/minecraft/variant-cits/item/goat_horn.json`
```json
{
	"type": "instrument",
	"modelPrefix": "item/goat_horn/"
}
```
These models should have a tooting override, just like the vanilla model.
Additional models for tooting should be stored in a separate directory: `item/tooting_goat_horn/`.
You can set your models' parents to the corresponding vanilla models, this will save you the need to redefine the "display" section.

## Music Discs
Relevant for datapacks that include custom songs.

All vanilla discs being different items, their are two ways to go about it. If you're confident your custom songs will only end up on a specific disc, you can create a module specific to it:

`/assets/minecraft/variant-cits/item/music_disc_<name>.json`
```json
{
	"type": "jukebox_playable",
	"modelPrefix": "item/music_disc_"
}
```

Otherwise you may create a single module encompassing all vanilla discs:

`/assets/minecraft/variant-cits/item/music_disc.json`
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
	"modelPrefix": "item/music_disc_"
}
```
Should a mod include custom disc types, those should not be included in the module for vanilla disc, but in a module unique to this mod.

Both modules the same format as the vanilla disc textures, and so will be compatible with each other.


## Potions
Being different items, splash potions and lingering potions require a different modules:

`/assets/minecraft/variant-cits/item/potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/potion/"
}
```

`/assets/minecraft/variant-cits/item/splash_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/splash_potion/"
}
```

`/assets/minecraft/variant-cits/item/lingering_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/lingering_potion/"
}

```
