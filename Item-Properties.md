# Item Properties

Item Properties are a type of parameter used by the `component_format` and `component_data` modules. They extract some data from an item stack, and return a string that the module will use to construct the variant id.

### Schema:
```json
{
	"property": "item_type",
	"fallback": "missingno",
	"...":"...",
	"transform": [ "lowercase", { "function":"regex", "...":"..." } ]
}
```
Additional fileds may be required depending on the property type, in particular the [`item_component`](#property-item_component) property, which your are most likely interested in.

#### Field: `property`
**Mandatory**, string.  
Technically optional and defaults to `"item_component"`, but remains mandatory for every other use.

Must be a [property type](#property-types) from the list below.

#### Field: `transform`
**Optional**, a single [transform](#transforms), or a array of transforms. Defaults to an empty array.

A series of transformations applied to the value of the property. Each transform in the array receives the string produced by the previous entry. The first transform receives the raw value of the property, and the output of the last transform is used as a building block for the variant ID.

#### Field: `fallback`
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
is short for:
```json
{
	"componentType": "custom_data",
	"nbtPath": ".key[1]",
	"expect": "auto",
	"transform": "sanitize_auto"
}
```


# Property types
## Property: `item_component`
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

#### Field: `componentType`
**Mandatory,** Namespaced identifier

The identifier of the item component to use.

#### Field: `nbtPath`
**Optional** String

The location of the data to extract within the input NBT. If left unspecified, the raw component will be used as the data.

- `.keyname` Expects a compound, and returns the value under the given name. "keyname" can contain any alpha-numerical characters, both upper and lower cases, as well as any character that is legal for a namespaced identifier to hold, excluding dots `'.'`.

- `[n]` expects an array, and returns the value at the given index. "n" must be a number. Negative values start at the end of the index.

- `{n}` expects a compound, but accesses it like an array of key-value pairs, and returns the pair at the given index. It should always be followed by either `.key` or `.value`.
The order in which entries are sorted is undefined.

> [!TIP]
> 
> In-game, you can use `/data get entity @s SelectedItem.components` to quickly check the nbt structure of an item in your main hand.

#### Field: `expect`
**Optional,** defaults to `"primitive"`. A single string, or an array of strings.

The expected data type at the end of the path, and how to interpret it.
All extracted data are converted to string, and then fed into the transforms.
Invalid data will be treated the same as if the data were missing.

If multiple types are specified, they will be tested in the provided order, until one of them successfully decodes the data.

Possible values are:
- **`auto`**: Tries to guess the correct data type. The exact implementation is experimental, and may receive change over time. You can use this for prototyping, but in order to future-proof your packs, you should use the correct type.
- **`string`**: Expects any string and extracts it as-is.
- **`number`**: Expects any numeric value. The type suffix is absent from the extracted string. (E.g: `1b` in the NBT will be interpreted as simply `1`)
- **`primitive`**: Expects either a string or a number.
- **`identifier`**: Expects a valid namespaced identifier. If the data has no namespace, `minecraft:` is made explicit in the extracted string.
- **`rich_text`**: Expects rich text (custom name), and extracts a plain text version, stripped of all formatting. You'll usually want to combine this with `"transform":"sanitize_path"` or similar.
- **`rich_text_array`**: Expects an array of rich texts (lore component). All entries are combined into a single string, adding a newline character (`'\n'`) after every entry.

## Property: `item_type`

Returns the identifier of the item type.

## Property: `item_count`

Return the exact count of the item stack.

> [!CAUTION]
>
> This property is a stub and will not behave intuitively. Unlike the `item_count` module, this will only match CITs to stacks that have **exactly** the correct count, as opposed to equal-or-higher.

## Property: `axolotl_variant`

Returns the color name of an axolotl in a bucket.

The nbt representation of this data varies between versions of minecraft, so using this property can be more reliable than `item_component`.

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

The nbt representation of this data varies between versions of minecraft, so using this property can be more reliable than `item_component`.

# Transforms
A transform is simply a function that takes in a string (the value of a property), and outputs a different strings (then used in the variant id).
Transforms are usually wrapped in an array, all transforms in the array will be chained together to produce more complex results.

Some transforms can fail to produce an output, in which case the property will be considered invalid by the module.
## Schema
```json
{
	"function": "transform_type",
	"optional": true,
	"fallback": "fallback_value",
	"...": "..."
}
```
Additional fields may be required depending on the transform type.

If a transform takes no parameter other than its type, the whole schema can be compressed into a single string.
For example, these two codes behave identically:
```json
"transform": [
	{ "function": "lowercase" },
	{ "function": "discard_namespace" },
]
```
```json
"transform": [
	"lowercase",
	"discard_namespace"
]
```

### Field: `function`
**Mandatory**, string.  
Technically optional and defaults to `"regex"`, but is mandatory on everything other type.

Must be a [transform type](#transform-types) from the list below.

### Field: `optional`
**Optional**, boolean, defaults to `false`.

If this is set to true, a transform that would fail will instead return its input, unmodified.

### Field: `fallback`
**Optional**, string.

If this is set, a transform that would fail will instead return the given value.
This option overrides `optional`.

# Transform types
## Simple transforms
These transforms take no parameter and can never fail. They're almost always specified as plain string.
- **`lowercase`**: Converts all upper-case characters to lower-case.
- **`sanitize`**: Removes all characters that are illegal for an identifier. Uppercases are replaced with lowercases, spaces are replaced with underscores, accentuated characters have their accents stripped, and all other invalid characters are removed completely.
**The result is not guaranteed to be a valid identifier**; for example it may still contain more than one column `':'`, or the namespace could contain a slash `'/'`.  
See also: [`charset_remap`](#transform-charset_remap).
- **`sanitize_path`**: Similar to `sanitize`, but removes all `':'`, and so always returns a valid identifier path.
- **`sanitize_namespace`**: Similar to `sanitize`, but removes all `':'` and `'/'`, and so always returns a valid identifier namespace.
- **`sanitize_auto`**: Does nothing if the string is a valid identifier. Otherwise, behaves identically to `sanitize_path`.
- **`discard_namespace`**: Removes `':'` and all preceding characters. Behaviour is undefined on strings that contain multiple columns.
- **`discard_path`**: Removes `':'` and all following characters. Behaviour is undefined on strings that contain multiple columns.

## Transform: `regex`
Makes use of **Regular Expressions** to either modify, or invalidate the its input.

If the input matches the regex, it will be replaced with the substitution string.

> [!TIP]
>
>  Use [regex101.com](https://regex101.com/) to test you patterns and substitution strings.

### Schema
```json
{
	"regex": "(?i)Prefix_(.*)",
	"substitution": "$1",
	"matchAll": true,
}
```
> Specifying `"function":"regex"` is optional only for this particular transform type.

In this example, the regex asserts that the input must start with `"prefix_"`, and removes that prefix from the string.  

`(?i)` means this is case-insensitive.   
`.*` matches any string.  
`(.*)` is a **capture group** that grabs the unprefixed string.  
`$1` returns the value of the first capture group  


#### Field: `regex`
**Mandatory** string.

The pattern the property should match.
Regex flags can be defined at the start of the pattern using inline modifiers, such as `(?imsxu)`

#### Field: `substitution`

**Optional** string, defaults to `"$0"` (no-op).

Constructs a new value for the property from values captured by the pattern. The default value passes the original string through unmodified.

#### Field: `matchAll`

**Optional** boolean, defaults to `true`.

Whether the regex should match the entire string, or only subsections of it. 
Eg: This changes whether the regex `"b"` will match the string `"abc"`. Setting this to `false` with `"optional":true` can be used as a "search and replace" mode.


## Transform: `test`
Wraps another transform, and checks whether it fails.
If the inner transform fails, this also fails.
If the sub-transform *succeeds*, then the *original* value is returned instead of the transformed value.
### Schema
```json
{
	"function": "test",
	"tester": [ 
		"lowercase",
		{ "function":"whitelist", "whitelist":["..."] }
	]
}
```
In this example, the whitelist will evaluate a lowercase string, but the capitalized string will be returned on success.

## Transform: `alternative`
Contains multiple chains of transforms, and tests them independently until *one of them* succeeds. This returns the result of the first transform to succeed. This fails if all sub-transforms fail.
###	Schema
```jsonc
{
	"function": "alternative",
	"alternatives": [
		// First possibility
		[ "lowercase", {"regex":"..."} ], 
		// Second possibility
		"sanitize_auto",
		// Etc
	]
}
```

Unlike every other place where arrays of transforms are used. The root array **is not** a chain of transform. It is the entries that can be chains of transforms.

## Transform: `whitelist` / `blacklist`
Fails depending on whether the input is in the given list.
The original value is returned in case of success.
###	Schema
```json
{
	"function": "whitelist",
	"whitelist": [
		"allowed_value_1",
		"allowed_value_2",
		"allowed_value_3",
	]
}
```
```json
{
	"function": "blacklist",
	"blacklist": [
		"disallowed_value_1",
		"disallowed_value_2",
		"disallowed_value_3",
	]
}
```

## Transform: `remap`
If the input value is in the provided map's keys, this will return the associated value instead.
This fails if the input is not in the map.
```json
{
	"function": "remap",
	"map": {
		"input_1": "output_1",
		"input_2": "output_2",
		"...": "..."
	}
}
```

## Transform: `charset_remap`

Individually replaces or delete characters in the input string.
This can, for example, be used prior to `sanitize`, in order to preserve meaningful characters that would otherwise be removed.

This transform never fails.

### Schema
```json
{
	"function": "charset_remap",
	"source":      "абвгдезийклмнопрстуфхюя",
	"destination": "abvgdezijklmnoprstufxyy",
	"delete":      "ь",
	"map": {
		"ж": "zh",
		"ц": "ts",
		"ч": "ch",
		"ш": "sh"
	}
}
```
> ⚠ I am not familiar with the cyrillic alphabet. Take this example with a grain of salt.

#### Field: `source`, `destination`
**Mandatory,** *Strings*

Each character present in `source` will be replaced with the one of equivalent index in `destination`.

If `source` is longer than `destination`, superfluous characters will be deleted instead of replaced.
Behaviour is undefined if `source` contains multiple occurences of the same character.

#### Field: `delete`
**Optional**, *String*

Characters listed here will be deleted instead of replaced. This overrides mappings done in `source` and `destination`.

#### Field: `map`
**Optional**, *Maps Characters to Strings*

`map` behaves the same the same as the three previous fields, but allows one character to be mapped to a longer string.

This overrides mappings done in the previous fields.
