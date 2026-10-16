# Tutorial: Getting Started (with `predicates`)
## Prerequisites
This tutorial assumes that you are at least familiar with JSON syntax.  
No other knowledge of the mod is needed.

## Foreword
This tutorial focuses exclusively on the `predicates` module type. It works nothing like the other types, but its is probably the most beginner-friendly.
This type is one of the most recent additions to the mod, so as of writing, you will likely not find many packs out there that rely on it.

## Creating a module
A **Module** is a file in charge of changing the model of an item. If you are familiar with Optifine, it is basically like a CIT file, but with one major difference: Optifine files only manage a single model each, whereas Modules can manage entire collections of them.  
On that aspect, modules are closer to how Minecraft's own item models work.

Create a JSON file somewhere in: `assets/<namespace>/variants-cit/modules/<path>`.  
The filename have no bearing on the module's functionality.
The name needs to be unique, or other packs could overwrite your module with their own. Putting your module in a custom namespace is good practice to avoid that.
The filename sill needs to abide by Minecraft's rule on what constitutes a [valid path](https://minecraft.wiki/w/Identifier#Legal_characters). VCIT doesn't make any accomodation for illegal pathes like Optifine does.

```json
{
	"item": "diamond_sword",
	"assetGen": "item_model/handheld",

	"type": "predicates",
	"parameters": {
		"predicates": []
	}
}
```
The module above is currently empty and will not do anything, this is only the baseline before adding actual variants.


The detailed list of options for a module is listed [here](./Module-Configuration), but here's a quick rundown of the important ones here :

- **`item`** is the affected item types. It can be an array containing multiple item types.

- **`assetGen`** tells the module how to generate JSON models for your textures. Typical values are `item_model/generated` for regular items, and `item_model/handheld` for tools. More advanced items like bows, tridents, etc, have their own setting; you will find the full list [here](./Asset-Generation#built-in-asset-generator-presets).


## Adding Variants
Variants of your item go inside `parameters{ "predicates":[] }`.
You can add as many of these as you want inside the same module. 
Predicates are evaluated top to bottom, so the topmost has the highest priority.


```json
"parameters": {
	"predicates": [
		{
			"modelId": "custom_swords/epee_de_lend",
			"precondition": {
				"display_name": "Épée de l'End"
			}
		}
	]
}
```

- **`modelId`** is the name of the model to use.  
For instance, the model ID `custom_swords/epee_de_lend` will use the texture at:  
`assets/minecraft/textures/item/custom_swords/epee_de_lend.png`  
Model IDs can of course still use custom namespaces.

- **`precondition`** is the set of conditions required for the model to apply.  
  - The names on the left (here "`display_name`") can be the name of any [vanilla component](https://minecraft.wiki/w/Data_component#List_of_components), or some other [property](./Item-Properties) provided by the mod.
  - The values on the right is a [chain of transforms](./Transforms) that tests the value extracted from the item.

Transforms are basically functions, that takes in data from the item, and outputs a different piece of data. If you are familiar with Minecraft's Loot functions, chains of transforms use a similar syntax. The condition is met if the transform is able to output *any* data.  
Here, `"Épée de l'End"` is short for `{"equals":"Épée de l'End"}`, which simply checks the input as the expected value.

## Other examples of condition

Have a look at the precondition [**cheat-sheet**](./Precondition%20Cheat-Sheet#cheat-sheet) for a quick run-down of what can be done with these. You can also look through [issues labelled `precondition`](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3Aprecondition) to see what other people have done or needed.


### Pattern Matching / Case insensitive Matching
Using a plain string as the condition for a name will only look for an exact match. In the example module above, `"display_name":"Épée de l'End"` will not match items named "épée de l'end" or "Epee de l'End".

For looser matching, use **Regular Expressions**:
```jsonc
"display_name": { "regex": "(?iu)[ÉE]p[ée]e de l'End" }
```
- '`(?iu)`' makes matching case-insensitive.
- '`[ÉE]`' matches a single E with or without accent.

Wildcards are done using '`.*`'. You could for example add one at the beginning to match an optional possessive player name. (E.g: "Steve's Epee de l'end.")
```jsonc
"display_name": { "regex": "(?iu).*[ÉE]p[ée]e de l'End" }
```

To experiment with regular expressions, use [Regex101](https://regex101.com/).

### Checking number thresholds
Using a plain number as the condition once again only looks for an exact match
```jsonc
"enchantment.sharpness": 1 // Exact match
"enchantment.sharpness": { "equals": 5 } // Also exact match
"enchantment.sharpness": { "greater_or_equals": 5 }
"enchantment.sharpness": { "smaller_or_equals": 5 }
"enchantment.sharpness": { "greater_than": 5 }
"enchantment.sharpness": { "smaller_than": 5 }

// Shorthand for checking an enchantment exists at any level
"enchantment.sharpness": []
// Enchantment must be ABSENT from the item
"!enchantment.sharpness": []
```

Using `{ "equals": 0 }` will NOT work for the purpose of testing an enchantment is absent. The NBT data does not actually contain entries for enchantment that are absent, so there is no data to be compared against 0, and the condition would fail because it cannot find the data.

### Be ingenious !

Transforms can both test and modify data, and can be chained together to test the modified data instead. Other transforms such as `foreach` and `alternative` can combine other transforms in more complex ways, such as testing every entries in a list:

```jsonc
// The item must have no enchantment except curses
"enchantments": {
	"function": "foreach",
	// Iterates over enchantment names
	"foreach": "key",
	// Test the whitelist against each enchantment name
	"do": { "whitelist_id": [
		"vanishing_curse",
		"binding_curse"
	]},
	// All enchantments on the item must belong to this list.
	"return": "matches_all"
}
```

Again, check [**here**](./Transforms) for the full list of transforms and what they do.

## Quick Troubleshooting

You can find a list of common issues here: [**Troubleshooting**](./Troubbleshooting)  
If there is *one* thing you need to take away from that page, it's that these two commands exist. Use them without reserve: 

```
/variants-cit module <hook> <module id> walkthrough
/variants-cit moduletree <hook> crawl
```
"**Crawl**" is for figuring out which module applied to an item.  
"**Walkthrough**" is for figuring out why a given module did or did not apply to an item.  
Both commands run on the item in your main hand by default.

`<hook>` will be "`item_model`" unless your module declares another one.  
`<module id>` depends on the filename of your module.  

> [!TIP]
>
> You can use `F3 + D` to clear the in-game chat. For commands that give a lot of feedbacks, this will make it easier to find where things start.


## Read next:
- [**Tutorial: Understanding Asset-Generation**](./Understanding-AssetGen) (Using custom JSON model)
- [**Tutorial: Changing the look of Equipped Armor**](./Equipped-Armor)

## See Also:
- [**Tutorial: Pattern-Seeking modules**](./Seeking-Patterns)  
  Do you have several dozens or even hundreds variants to your items ? Do they follow a pattern ? Have you ever wished you didn't need to manually write down each and every one of these into a file ?

