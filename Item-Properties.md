# Item Properties

Item Properties are a type of parameter used by the `component_format` and `component_data` modules. They extract some data from an item stack, and return a string that the module will use to construct the variant id.

### Schema:
```json
{
	"property": "item_type",
	"transform": "discard_namespace",
	"fallback": "missingno"
}
```

Some property types may take additional parameters, in particular the [`item_component`](#item_component) property, which your are most likely interested in.

#### `property`
**Optional**, defaults to `"item_component"`

See [Property Types](#property-types) below for a list of possible properties and their parameters.

#### `transform`
**Optional**, a single [transform](#transforms), or an array of transforms. Defaults to an empty array.

A series of transformations applied to the value of the property.

#### `fallback`
**Optional** string.

If specified, the property will never fail.
If the property is missing or invalid, this string will be used as the property's value instead.

### Simplified formatting
Some properties can be condensed into a single string.

For properties with no mandatory parameters, just give the property name: `"item_type"` is short for `{"property":"item_type"}`.

For `item_component` properties, instead provide the component's name, immediately followed by the optional nbt path:
```json
"custom_data.key[1]"
```
will be expanded to:
```json
{
	"componentType": "custom_data",
	"nbtPath": ".key[1]",
	"expect": "auto",
	"transform": "sanitize_auto"
}
```

## Transforms
Transforms can be specfied as either a single value/object, or as an array of them.
When providing an array, each transform receives the result of the previous transform.

### Simple transforms
Simple transforms take no parameters and are specified as plain string:
- **`lowercase`**: Converts all upper-cases to lower-cases.
- **`sanitize`**: Removes all characters that are illegal for an identifier. Uppercases are replaced with lowercases, spaces are replaced with underscores, accentuated characters have their accents stripped, and all other invalid characters are removed completely.
**The result is not guaranteed to be a valid identifier**; for example it may still contain more than one column `':'`.
- **`sanitize_path`**: Like `sanitize`, but also strips all characters that are illegal for an identifier's path.
- **`sanitize_namespace`**: Like `sanitize`, but also strips all characters that are illegal for an identifier's namespace.
- **`sanitize_auto`**: Does nothing if the string is a valid identifier. Otherwise, behaves like `sanitize_path`.
- **`discard_namespace`**: Removes `':'` and all preceding characters. Behaviour is undefined on strings that contain multiple columns.
- **`discard_path`**: Removes `':'` and all following characters. Behaviour is undefined on strings that contain multiple columns.

### Regex Transform
Regex transform makes use of *regular expressions* to either modify, or invalidate the value of a property.

This transform takes parameters, and so must be specified as an object:

```json
{
	"regex": "(?i)Prefix_(.*)",
	"substitution": "$1",
	"matchAll": true,
	"validate": false
}
```

#### `regex`
**Mandatory** string.

The pattern the property should match.
Regex flags can be defined at the start of the pattern using inline modifiers, such as `(?imsxu)`

#### `substitution`

**Optional** string, defaults to `"$0"` (no-op).

Constructs a new value for the property from values captured by the pattern. The default value passes the original string through unmodified.

#### `matchAll`

**Optional** boolean, defaults to `true`.

Whether the regex should match the entire string, or only subsections of it. Eg: whether the regex `"b"` will match the string `"abc"`. Setting this to false can be used as a "search and replace" mode.

#### `validate`
**Optional** boolean, defaults to `true`.

What to do if the regex does not match the string. If false, the original string is passed through unmodified. If true, the data is considered invalid, and the module will not be applied to the item.

> [!TIP]
>
>  Use [regex101.com](https://regex101.com/) to test you patterns and substitution strings.


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
**Optional** String

The location of the data to extract within the input NBT. If left unspecified, the raw component will be used as the data.

- `.keyname` Expects a compound, and returns the value under the given name. "keyname" can contain any alpha-numerical characters, both upper and lower cases, as well as any character that is legal for a namespaced identifier to hold, excluding dots `'.'`.

- `[n]` expects an array, and returns the value at the given index. "n" must be a number. Negative values start at the end of the index.

- `{n}` expects a compound, but accesses it like an array of key-value pairs, and returns the pair at the given index. It should always be followed by either `.key` or `.value`.
The order in which entries are sorted is undefined.

> [!TIP]
> 
> In-game, you can use `/data get entity @s SelectedItem.components` to quickly check the nbt structure of an item in your main hand.

#### `expect`
**Optional,** defaults to `"primitive"`. A single string, or an array of strings.

The expected data type at the end of the path, and how to interpret it.
All extracted data are converted to string, and then fed into the transforms.
Invalid data will be treated the same as if the data were missing.

If multiple types are specified, they will be tested in the provided order, until one of them successfully decodes the data.

Possible values are:
- **`auto`**: Tries to guess the correct data type. The exact implementation is experimental, and may receive change. Use this for quick testing, but for future-proofing, keep using the correct type.
- **`string`**: Expects any string and extracts it as-is.
- **`number`**: Expects any number type. The type suffix is absent from the extracted string. (E.g: `1b` in the NBT will be interpreted as simply `1`)
- **`primitive`**: Expects either a string or a number.
- **`identifier`**: Expects a valid namespaced identifier. If the data has no namespace, `minecraft:` is made explicit in the extracted string.
- **`rich_text`**: Expects rich text, and extracts a plain text version, stripped of all formatting. You'll usually want to combine this with `"transform":"sanitize_path"` or similar.
- **`rich_text_array`**: Expects an array of rich texts. All entries are combined into a single string, adding a newline character (`'\n'`) after every entry.

## `item_type`

Returns the identifier of the item type.

## `item_count`

Return the exact count of the item stack.

> [!CAUTION]
>
> This property is a stub and will not behave intuitively. Unlike the `item_count` module, this will only match CITs to stacks that have **exactly** the correct count, as opposed to equal-or-higher.

## `axolotl_variant`

Returns the color name of an axolotl in a bucket.

The nbt representation of this data varies between versions of minecraft, so using this property is more reliable than `item_component`.

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

The nbt representation of this data varies between versions of minecraft, so using this property is more reliable than `item_component`.
