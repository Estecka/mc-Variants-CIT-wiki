# Recommended Configurations

This mod only supports a single CIT config per item type. If multiple resource packs want add different variants to the same item, it's important that they use similar enough config in order to be able to cooexist.
In particular, it's critical that the model prefix be identical for all; as a rule of thumbs, you should use the item's name as the variant directory.

Here is a collection of modules I encourage you to use as-is.

## Axolotl Buckets
`/assets/minecraft/variants-cit/item/axolotl_bucket.json`
```json
{
	"type": "axolotl_variant",
	"modelPrefix": "item/axolotl_bucket_"
}
```
This is the same format used by the mod "More Axolotl Variants"

## Enchanted Books
`/assets/minecraft/variants-cit/item/enchanted_book.json`
```json
{
	"type": "stored_enchantments",
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
`/assets/minecraft/variants-cit/item/goat_horn.json`
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
The same config should be used for every music disc types. Although in practice you'll only need to provide configs for the variants you use to host you custom discs.

`/assets/minecraft/variants-cit/item/music_disc_<host>.json`
```json
{
	"type": "jukebox_playable",
	"modelPrefix": "item/music_disc_"
}
```
This is the same format that is used in the vanilla resources.

## Potions
Being different items, splash potions and lingering potions require a different CIT config.

`/assets/minecraft/variants-cit/item/potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/potion/"
}
```

`/assets/minecraft/variants-cit/item/splash_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/splash_potion/"
}
```

`/assets/minecraft/variants-cit/item/lingering_potion.json`
```json
{
	"type": "potion_type",
	"modelPrefix": "item/lingering_potion/"
}

```