# Item Properties

Item Properties extract some data from an item stack. The data can then be used as is, or sent into a [Transform](./Transforms) to either, depending on the context, check that it meets some conditions, or modify the value before usage.

They can be used in every module's [preconditions](./Precondition%20Cheat-Sheet), and in the parameters of `component_data`, `component_format`, and `predicates` module types.

## Index
- [Schema](#schema)
- Property Types:
	- [`item_component`](#property-item_component)
	- [`item_count`](#property-item_count)
	- [`item_type`](#property-item_type)
	- [`axolotl_variant`](#property-axolotl_variant)
	- [`bucket_entity_age`](#property-bucket_entity_age)
	- [`display_name`](#property-display_name)
	- [`painting_variant`](#property-painting_variant)

## Schema:
```json
{
	"property": "display_name",
	"transform": [
		{ "function": "regex", "regex":"..." },
		{ "function": "sanitize_path" },
		{ "function": "get_identifier", "defaultNamespace": "..." }
	],
	"fallback": "missingno"
}
```
Additional fields may be required depending on the property type, in particular the [`item_component`](#property-item_component) property, which your are most likely interested in.

### Field: `property`
**Mandatory**, Identifier.  
Technically optional and defaults to `"item_component"`, but remains mandatory for every other use.

Must be a [property type](#property-types) from the list below.

### Field: `transform`
**Optional**, a [chain of transforms](./Transforms). Defaults to an empty array.

A series of functions applied to the value of the property.
Multiple transforms can be chained together to produce more complex results.

### Field: `fallback`
**Optional** string or number.

If specified, the property will never fail. If the property is missing, or if the transform failed to process the data, then the fallback value will be used instead.


# Property types
## Property: `item_component`
Use any single component as a source of data.
If `nbtPath` is specified, the component is converted to NBT, and the NBT element at the end of the path is extracted.

### Schema:

```json
{
	"componentType": "fireworks",
	"nbtPath": ".somearray[0].somemap{0}.key"
}
```
> Specifying `"property":"item_component"` is optional only for this particular property.

#### Field: `componentType`
**Mandatory,** identifier

The identifier of the item component to use.

#### Field: `nbtPath`
**Optional** String

The location of the data to extract within the input component.

If left unspecified, the raw component will be used as the data. Otherwise, the component is converted to NBT, and the property will output the nbt element at the end of the path.

This field behaves identically to its transform counterpart. See: [Transform: `nbt_path`](./Transforms#transform-nbt_path) for the details on the syntax.

> [!TIP]
> 
> In-game, you can use `/data get entity @s SelectedItem.components` to quickly check the nbt structure of an item in your main hand.
> 
> Alternatively, you can use the [`walkthrough`](./Troubbleshooting#command-walkthrough) command, which will print any component used by a module.


## Property: `item_type`

Returns the identifier of the item's type.

This property takes no parameter.


## Property: `item_count`

Return the exact count of the item stack.

This property takes no parameters.

> [!IMPORTANT]
>
> When used in `component_data` or `component_format`, the module will match a model to an item stack only if they have **exactly**. Using the item count as a threshold value will only work in `predicates` or `item_count` modules, or inside the Preconditions of a module.
>

## Property: `axolotl_variant`

Returns the color name of an axolotl in a bucket.

The component representing this data varies between versions of minecraft, so using this property can be more reliable than `item_component`.


## Property: `display_name`

Returns the effective name of the item; whatever is displayed in the item's tooltip.

Minecraft determines the display name using these sources, in order:
1. The `custom_name` component
2. Hardcoded name variations (Lodestone compass, banner shields)
3. The `item_name` component


## Property: `bucket_entity_age`

Returns either of two strings, depending on whether the entity in a bucket is an adult (age >= 0) or a baby (age < 0).

### Schema:
```json
{
	"property": "bucket_entity_age",
	"adult": "",
	"baby": "_baby"
}
```
#### Field: `adult`
**Optional**, defaults to an empty string.

The value to return for adults.

#### Field: `baby`
**Optional**, defaults to `"_baby"`.

The value to return for babies.

## Property: `painting_variant`

Returns the variant of a painting picked from the creative inventory.

The component representing this data varies between versions of minecraft, so using this property can be more reliable than `item_component`.

# Transforms

**MOVED TO IT'S OWN PAGE: [TRANSFORMS](./Transforms)**
