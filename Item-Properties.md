# Item Properties

Item Properties are a type of parameter used in the `component_format` and `component_data` module. They extract some data from an item stack, and return a string that the module will use to construct the variant id.

### Schema:
```json
{
	"property": "item_type",
	"transform": "discard_namespace"
}
```

Some property types may take additional parameters.

#### `property`
**Optional**, defaults to `"item_component"`

See [Property Types](#property-types) below for a list of possible properties and their parameters.

When a property has no mandatory parameters  `{"property":"item_type"}` can be condensed to just `"item_type"`.

#### `transform`
**Optional**, a single string, or an array of strings. Defaults to an empty array.

A series of transformations applied to the value of the property. The order of the items in the array matters.

Possible transformations are:
- **`lowercase`**: Converts all upper-cases to lower-cases.
- **`sanitize`**: Removes all characters that are illegal for an identifier. Uppercases are replaced with lowercases, spaces are replaced with underscores, accentuated characters have their accents stripped, and all other invalid characters are removed completely.
**The result is not guaranteed to be a valid identifier**; for example it may still contain more than one column `':'`.
- **`sanitize_path`**: Like `sanitize`, but also strips all characters that are illegal for an identifier's path.
- **`sanitize_namespace`**: Like `sanitize`, but also strips all characters that are illegal for an identifier's namespace.
- **`discard_namespace`**: Removes `':'` and all preceding characters. Behaviour is undefined on strings that contain multiple columns.
- **`discard_path`**: Removes `':'` and all following characters. Behaviour is undefined on strings that contain multiple columns.

# Property types
## `item_component`
Extracts data from any component, using that component's NBT representation.

If the component or the data is missing, or the data does not have the expected type, the module will not apply to the item stack.

### Schema:

```json
{
	"componentType": "fireworks",
	"nbtPath": ".array[0].map{0}.key",
	"expect": "rich_text"
}
```

Specifying `"property":"item_component"` is optional only for this particular property.

#### `componentType`
**Mandatory,** Namespaced identifier

The identifier of the component to explore.

#### `nbtPath`
**Mandatory,** String

The location of the data to extract within the input NBT. If the data is the root NBT itself, the path will be an empty string.

- `.keyname` Expects a compound, and returns the value under the given name. "keyname" can contain any alpha-numerical characters, both upper and lower cases, as well as any character that is legal for a namespaced identifier to hold, excluding dots `'.'`.

- `[n]` expects an array, and returns the value at the given index. "n" must be a number. Negative values start at the end of the index.

- `{n}` expects a compound, but accesses it like an array of key-value pairs, and returns the pair at the given index. It should always be followed by either `.key` or `.value`.
The order in which entries are sorted is undefined.

> [!TIP]
> 
> In-game, you can use `/data get entity @s SelectedItem.components` to quickly check the nbt structure of an item in your main hand.

#### `expect`
**Optional,**  defaults to `"primitive"`.

The expected data type at the end of the path, and how to interpret it.
All extracted data are converted to string, and then fed into the transforms.
Invalid data will be treated the same as if the data were missing.

Possible values are:
- **`string`**: Expects any string and extracts it as-i**s**.
- **`number`**: Expects any number type. The type suffix is absent from the extracted string. (E.g: `1b` in the NBT will be interpreted as simply `1`)
- **`primitive`**: Expects either a string or a number.
- **`identifier`**: Expects a valid namespaced identifier. If the data has no namespace, `minecraft:` is made explicit in the extracted string.
- **`rich_text`**: Expects ich text, and extracts a plain text version, stripped of all formatting. You'll usually want to combine this with `"transform":"sanitize_path"` or similar.

## `item_type`

Returns the identifier of the item type.

## `item_count`

Return the exact count of the item stack.

> [!CAUTION]
>
> This property is a stub and will not behave intuitively. Unlike the `item_count` module, this will only match CITs to stacks that have **exactly** the correct count, as opposed to equal-or-higher.

## `axolotl_variant`

Returns the color name of an axolotl in a bucket.

The nbt representation of this data varies between version of minecraft, so using this property is more reliable than `item_component`.

## `bucket_entity_age`

Returns either of two strings, depending on whether the entity in a bucket is an adult (age >= 0) or a baby (age < 0).

### Schema:
```json
{
	"property": "bucket_entity_age",
	"adult": "",
	"baby": "_baby"
}
```
#### `adult`
**Optional**, defaults to an empty string.

The value to return for adults.

#### `baby`
**Optional**, defaults to `"_baby"`.

The value to return for babies.

## `painting_variant`

Returns the variant of a painting from the creative inventory.

The nbt representation of this data varies between version of minecraft, so using this property is more reliable than `item_component`.
