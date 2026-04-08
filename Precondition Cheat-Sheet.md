# Preconditions
Conditions have several syntaxes.

The simplified syntax is easier on the eyes and much faster to type, but can create ambiguity in some extreme edge cases.

The canonical syntax is more verbose, but stricter. If you are already familiar with `component_data`, this is a similar syntax. Learning to use preconditions through the lens of the canonical syntax will make it easier to understand how conditions relate to [item properties](./Item-Properties).

It is possible to mix and match different syntaxes at different levels as needed.

> [!CAUTION]
> 
> This feature is experimental, its syntax may receive breaking changes.
>
> The canonical syntax is pretty much set in stone, but the simplified syntax is likely to receive adjustments. JSON is not a format that lends itself very well to this kinds of syntax. Suggestions are welcome.

## Canonical Syntax
There are only 3 types of conditions: `matches_any`, `matches_all` and `transform`. The later is where all the interesting stuff happens. The others are just wrappers for other conditions.

### Example:
```jsonc
// if (A and B and (C or D))
"precondition": {
	"condition": "matches_all",
	"all":
	[
		{
			"condition": "transform", // A
			"componentType": "custom_data",
			"nbtPath": ".sword_type",
			"transform": { "equals": "END_SWORD"}
		},
		{
			"condition": "transform", // B
			"negate": true,
			"property": "item_count",
			"transform": { "greater_than": 1 }
		},
		{
			"condition": "matches_any",
			"any":
			[
				{
					"condition": "transform", // C
					"componentType": "custom_name",
					"transform": { "regex": "(?i)Ascended .*"}
				},
				{
					"condition": "transform", // D
					"componentType": "item_name",
					"transform": { "regex": "(?i)Ascended .*"}
				}
			]
		}
	]
}
```

All conditions takes a `condition` and a `negate` field.

### Field: `condition`
**Mandatory** String

One of the condition type described below.

### Field: `negate`
**Optional** boolean, defaults to `false`.

If enabled, the result of the condition will be inverted.

### Condition: `matches_any`, `matches_all`
Each have single field called either `any` or `all`, accordingly. It is an array of other conditions.

### Condition: `transform`
Makes use of [item properties](./Item-Properties#item-properties) and [transforms](./Item-Properties#transforms) to validate the value of some data. If the transform succesfully produces a result, the condition passes. If no transform is specified, this simply checks that the property exists on the item.


## Simplified syntax
The example above can be simplified to this:
```jsonc
// if (A and B and (C or D))
"precondition":
{
	"custom_data.sword_type": "END_SWORD", // A
	"!item_count": { "greater_than": 1 }, // B
	"matches_any":
	{
		"custom_name": { "regex": "(?i)Ascended .*" }, // C
		"item_name":   { "regex": "(?i)Ascended .*" } // D
	}
}
```

The value of `matches_any` can be represented as map instead of an array, where each entry is a condition. When listed inside an array, such a map will always be treated as a `matches_all` condition.

If an entry's key starts with an exclamation mark, the condition is negated.

The fields `matches_any` and `matches_all` are treated as the corresponding condition type. Every other entry are be interpreted as a `transform` condition:

Keys of the map can be either the ID of an [item property](./Item-Properties#property-types) with no parameters, or an [`item_component`](./Item-Properties#property-item_component) property represented by the value of its `componentType` and its `nbtPath`.

The values of the map are the transform chains associated with each property. If a value is a plain string or number, it will be interpreted as an [`equals`](./Item-Properties#transform-equals) transform.


In case a property provided by Variants-CIT has the same name as an item component (e.g: `axolotl_variant`), you can disambiguate between the two by using an explicit namespace: `variants-cit` or `minecraft`. By default, the property from the mod will be used.
Although I don't believe there is currently any reason to do so, this trick might end up being useful in the future.


## Cheat-Sheet
### Various types of requirements :
```jsonc
"precondition": {
	// Must be strictly equal
	"custom_data.path.to.data1": "somevalue",
	// Pattern matching
	"custom_data.path.to.data3": { "regex": "ultimate_.*" },
	// Case-insensitive matching
	"custom_name": { "regex": "(?i)Ultimate .*" },

	// Must be strictly equal
	"custom_data.path.to.data2": 20,
	// Enchantment must be present on the item.
	"enchantment.sharpness": { "greater_or_equals": 1 },
	// Enchantment must be absent from the item.
	"!enchantment.vanishing_curse": { "greater_than": 0 },

	// Component must exist. Any value allowed.
	// (Empty array = empty chain of transforms = no-op)
	"equippable": [],
	// Component must not exist.
	"!consumable": []
}
```

### Pitfalls:
```jsonc
"precondition": {
	// Enchantment level must be EXACTLY 1
	"enchantment.mending": 1,
	// In order to be equal to 0, the data MUST BE PRESENT in the nbt.
	"enchantment.vanishing_curse": 0,

	// Undefined behaviour. Do not use.
	"custom_data.eg1": true,
	"custom_data.eg2": false,
	"custom_data.eg3": null,
}
```

### Checking multiple requirements on the same data path
JSON does not support having multiple fields with the same name.
The following **is not valid** and will not work as expected:

```jsonc
// Undefined behaviour
{
	"enchantment.sharpness": { "greater_or_equals": 1 },
	"enchantment.sharpness": { "smaller_than": 5 }
}
```
Instead, use a `matches_any` or `matches_all` transform:

```jsonc
{
	"enchantment.sharpness": { "matches_any": [
		{ "smaller_than": 5 },
		{ "greater_or_equals": 1 }
	]}

}
```

### Checking whether data exists or not.
The downside of transforms is that they can only deal with data that exists. By themselves, they can't assert whether a piece of data exists.

An item property without any transform evaluates to true if any data exists.

```jsonc
// Canonical form
{
	"condition": "transform",
	"componentType": "equippable"
}
```
```jsonc
// Simplified form
{
	// Empty array = no-op transform
	"equippable": []
}
```

To require the data be absent, simply negate the above condition:
```jsonc
// Canonical form
{
	"condition": "transform",
	"componentType": "equippable",
	"negate": true
}
```
```jsonc
// Simplified form
{
	"!equippable": []
}
```

