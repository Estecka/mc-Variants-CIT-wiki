# Module Types

This page describes the behaviour of every built-in types to use in [module configurations](Module-Configuration).

Modules are designed around specific item types, but can still be used on any item, so long as they provides the proper component.

## `axolotl_variant`
Derives the item variant from the `Variant` element in the `bucket_entity_data` component. The component is used by multiple types of bucket-of-mob, but this CIT module only ever returns the ID of axolotl variants.

Axolotl variants are not inherently identifier-based, but this modules converts them to a namespaced identifier. It will ignore variants that are not stored in number format.

As of writting, available variant IDs are, in order: `lucy`, `wild`, `gold`, `cyan`, `blue`.

## `custom_data`
Derives the item variant from a string stored at the root of the `custom_data` component.
The key of the element this module will look for must be defined in its parameters.

### Parameters:
- `nbtKey`: *Mandatory, String.* The key of the element that will be treated as the variant ID.

Example:
```json
{
	"type": "custom_data",
	"modelPefix": "item/cutomItemType/",
	"parameters": {
		"nbtKey": "myNamespace:Variant"
	}
}
```

## `custom_name`
Derives the item variant from the `custom_name` component.

The name is directly used as the variant identifier, so most names should be restricted to the character set `[a-z0-9_.-]`.
Names with illegal characters can still be used, but the corresponding variant must be hardcoded in the module's parameters. Do note that only one resource pack in the stack can control that list of names.

### Parameters:
- `caseSensitive`: *Optional, Boolean, defaults to false.*
When false, all upper case characters are treated as lower cases. Note that uppercase are illegal characters for variant ids.
- `specialNames`: *Optional, Maps Strings to Identifiers.*
Hardcoded association between specific names and variant ID. Can be used with names that contain illegal characters.

## `instrument`
Derives the item variant from the item component `instrument`, used by goat horns.

## `jukebox_playable`
Derives the item variant from the item component `jukebox_playable`, used by music discs.

## `potion_type`
Derives the item variant from the item component `potion_contents`, used by potions, splash potions, and lingering potions. This specifically looks at the potion's "type", and not the actual status effects.

## `stored_enchantments`
Derives the item variant from the item component `stored_enchantments`, used by enchanted book.

If the item has **exactly one enchantments**, the variant will be the ID of that enchantment. If it contains multiple enchantments, it will instead use the special model `multi`.
