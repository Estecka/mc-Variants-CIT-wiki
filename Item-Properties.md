# Manipulating Item Data
### Outline
- Item Properties (Data Sources)
	- [`item_component`](#property-item_component)
	- [`item_count`](#property-item_count)
	- [`item_type`](#property-item_type)
	- [`axolotl_variant`](#property-axolotl_variant)
	- [`bucket_entity_age`](#property-bucket_entity_age)
	- [`painting_variant`](#property-painting_variant)
- Transforms (Data Functions)
	- [`alternative`](#transform-alternative)
	- [`charset_remap`](#transform-charset_remap)
	- [`discard_namespace`](#simple-transforms), [`discard_path`](#simple-transforms)
	- [`get_string`](#string-conversions), [`get_identifier`](#string-conversions), [`get_number`](#string-conversions), [`get_rich_text`](#string-conversions), [`get_rich_text_array`](#string-conversions)
	- [`lowercase`](#simple-transforms)
	- [`regex`](#transform-regex)
	- [`remap`](#transform-remap)
	- [`sanitize`](#simple-transforms), [`sanitize_path`](#simple-transforms), [`sanitize_namespace`](#simple-transforms), [`sanitize_auto`](#simple-transforms)
- Transforms (Predicates)
	- [`blacklist`](#transform-whitelist--blacklist)
	- [`equals`](#transform-equals)
	- [`regex`](#transform-regex)
	- [`greater_than`](#transform-smaller_than-smaller_or_equals-greater_than-greater_or_equal), [`greater_or_equals`](#transform-smaller_than-smaller_or_equals-greater_than-greater_or_equal)
	- [`smaller_than`](#transform-smaller_than-smaller_or_equals-greater_than-greater_or_equals), [`smaller_or_equals`](#transform-smaller_than-smaller_or_equals-greater_than-greater_or_equal)
	- [`test`](#transform-test)
	- [`whitelist`](#transform-whitelist--blacklist)


# Item Properties
Item Properties extract some data from an item stack. The data can then be used as is, or sent into a [Transform](#transforms) to either, depending on the context, check that it meets some conditions, or modify the value before usage.

They can be used in every module's [preconditions](./Precondition%20Cheat-Sheet), and in the parameters of `component_data`, `component_format`, and `predicates` module types.

### Schema:
```json
{
	"property": "item_type",
	"fallback": "missingno",
	"...":"...",
	"transform": [ "lowercase", { "function":"regex", "...":"..." } ]
}
```
Additional fields may be required depending on the property type, in particular the [`item_component`](#property-item_component) property, which your are most likely interested in.

#### Field: `property`
**Mandatory**, Identifier.  
Technically optional and defaults to `"item_component"`, but remains mandatory for every other use.

Must be a [property type](#property-types) from the list below.

#### Field: `transform`
**Optional**, a single [transform](#transforms), or a array of transforms. Defaults to an empty array.

A series of functions applied to the value of the property. The first transform receives the raw value of the property, then each transform in the array receives the string produced by the previous one.

#### Field: `fallback`
**Optional** string or number.

If specified, the property will never fail. If the property is missing, or if the transform failed to process the data, then the fallback value will be used instead.


# Property types
## Property: `item_component`
Extracts data from any component, using that component's NBT representation.
1. The specified is extracted from the item
2. If `nbtPath` is specified, the component is converted to NBT, and the NBT element at the end of the path is extracted.
3. If `expect` is specified, and the data is converted to that type.

### Schema:

```json
{
	"componentType": "fireworks",
	"nbtPath": ".somearray[0].somemap{0}.key",
	"expect": "rich_text"
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

Path syntax:
- **`.keyname`** or **`.'keyname'`** is used to access maps, it returns the value under the given key. "keyname" must be replaced with the actual name of the key.  
  The unquoted syntax supports a limited character set: `[a-zA-Z0-9:/_-]`.  
  The quoted syntax allows for any characters except single quotes and escape characters (`'` and `\`).  

  In both syntaxes, the escape character `\` can be used to intepret the next character literally, on the off-chance you do need a single quote in your keyname. Note that `\` is already an escape character in JSON, so you'll need to double them up everytime:
  1. Json file: `"nbtPath":".'weird\\'key\\\\"`
  2. As seen by the mod: `.'weird\'key\\'`
  3. Resulting key name: `weird'key\`

- **`[n]`** is used to access arrays, it returns the value at the given index. "n" must be a number. Negative values start at the end of the index.

- **`{n}`** is used to access maps in an array-like fashion, it returns the key-value pair at the given index. It should always be followed by `.key` or `.value` literally.
The order in which entries are sorted is undefined, but will remain constant for a given item.

> [!TIP]
> 
> In-game, you can use `/data get entity @s SelectedItem.components` to quickly check the nbt structure of an item in your main hand.

#### Field: `expect`
**Optional,** a single string, or an array of strings.

The expected data type(s). If the datacannot be converted to one of these types, the property will be considered invalid.

Possible values are **`number`**, **`string`**, **`identifier`**, **`rich_text`**, and **`rich_text_array`**. See the [String Conversion](#string-conversions) transforms below.

In most use cases, you will not need to specify an expected type; conversions to strings will occur automatically. This field may later be removed in favor of its transform counterpart.



## Property: `item_type`

Returns the identifier of the item's type.

This property takes no parameter.

## Property: `item_count`

Return the exact count of the item stack.

This property takes no parameters.

> [!IMPORTANT]
>
> When used in `component_data` or `component_format`, the module will match a model to an item stack only if they have **exactly**. Using the item count as a threshold value will only work in `predicates` or `item_count` modules, or inside the Preconditions of a module.

## Property: `axolotl_variant`

Returns the color name of an axolotl in a bucket.

The component representing this data varies between versions of minecraft, so using this property can be more reliable than `item_component`.

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
Transforms are functions that can be used to either modify or validate its input. All transforms can serve both roles; they will always either produce some piece of data (success) or produce nothing at all (failure).

Transforms are usually wrapped in an array, all transforms in the array will be chained together to produce more complex results.

Different transforms accept and produce different data types, usually strings or numbers. Most data types will implicitely be converted to strings for transforms that require it.

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

If a transform takes no parameter other than its type, its whole schema can be compressed into a string.
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
Must be a [transform type](#transform-types) from the list below.

This field is optional only for a select few types, which require only a single argument named after the transform itself: `regex`, `whitelist`, `blacklist`, `equals`, `smaller_than`, `smaller_or_equal`, `greater_than`, and `greater_or_equal`.

### Field: `optional`
**Optional**, boolean, defaults to `false`.

If this is set to true, this transform can never fail. When it would do so, it will instead return its input, unmodified.

### Field: `fallback`
**Optional**, string.

If this is set, this transform can never fail. When it would do so, it will instead return the given value.
This option overrides `optional`.

# Transform types
## Simple transforms
**Accepted input:** String

These transforms take no parameter. They accept strings, and will never fail when fed with this data type.
- **`lowercase`**: Converts all upper-case characters to lower-case.
- **`sanitize`**: Does nothing if the input is a valid identifier. Otherwise, behaves identically to `sanitize_path`. It is guaranteed to return a valid identifier.
- **`sanitize_legacy`**: The old behaviour of `sanitize`. Removes all characters that are illegal for an identifier. Uppercases are replaced with lowercases, spaces are replaced with underscores, accentuated characters have their accents stripped, and all other invalid characters are removed completely.
**The result is not guaranteed to be a valid identifier**; it may still contain more than one colon `':'`, or the namespace could contain a slash `'/'`.  
See also: [`charset_remap`](#transform-charset_remap).
- **`sanitize_path`**: Similar to `sanitize_legacy`, but also removes all `':'`, and so is guaranteed to return a valid identifier path.
- **`sanitize_namespace`**: Similar to `sanitize_legacy`, but also removes all `':'` and `'/'`, and so is guaranteed to return a valid identifier namespace.
- **`discard_namespace`**: Removes `':'` and all preceding characters. Behaviour is undefined on strings that contain multiple columns.
- **`discard_path`**: Removes `':'` and all following characters. Behaviour is undefined on strings that contain multiple columns.

## String Conversions
Most data types will automatically be converted to string, when fed into a transform that only accepts string. However, if that data is NBT (either because the component itself is NBT, or because it was explored using `nbtPath`), then there may exists multiple intepretations of it that are all valid, but will yield different results.

The transforms below are counterparts to the [`expect`](#field-expect) field, and behave exactly the same.
They do not take any parameters.

- **`get_string`**: Asserts that the input is a plain string.
- **`get_number`**: Asserts that the input is a plain number
- **`get_identifier`**: Asserts that the input is a plain string, and a valid identifier. If the namespace is missing from the input, "`minecraft:`" is made explicit in the output.
- **`get_rich_text`**: Asserts that the input is rich text (Eg: `custom_name` component).  
  If the input is a plain string, this will still attempt to convert it to rich text, which could fail, or result in a different string.  
  When rich text is converted to plain strings, it is stripped of its formatting entirely.  
  If your text is used as the variant of a `component_format` or `component_data` module, you'll usually want to combine it with a `sanitize` transform or similar.
- **`get_rich_text_array`**: Asserts that the input is an array of rich texts. (Eg: `lore` component)  
  When converted to a plain strings, entries are combined together. A newline character (`'\n'`) is inserted after every entry, including the final line.

## Transform: `regex`
**Accepted input:** String

Makes use of **Regular Expressions** to either modify, or validate the string.
If the input matches the regex, it will be replaced with the substitution string if specified

> [!TIP]
>
> Use [regex101.com](https://regex101.com/) to test you patterns and substitution strings.  
> Make sure to set the regex flavor to "Java" in the left panel.

### Schema
```json
{
	"regex": "(?i)Prefix_(.*)",
	"substitution": "$1",
	"matchAll": true,
	"multiline_handling": "first_match_only"
}
```
> Specifying `"function":"regex"` is optional for this transform.

In this example, the regex asserts that the input must start with `"prefix_"`, and removes that prefix from the string.  

#### Crash-course on regex syntax
In the example above:
- `(?i)` means matching is case-insensitive.
- `.*` matches any string.
- `(.*)` is a **capture group** that grabs the unprefixed string.
- `$1` returns the value of the first capture group

#### Field: `regex`
**Mandatory** string.

The pattern the property should match.
Regex flags can be toggled inside the pattern itself, using inline modifiers, such as `(?imsxu)`

#### Field: `substitution`
**Optional** string, defaults to `"$0"` (no-op).

The value to return, which can include capture groups used in the pattern. The default value passes the original string through unmodified.

#### Field: `matchAll`
**Optional** boolean, defaults to `true`.

Whether the regex should match the entire string, or only subsections of it. 
Eg: This changes whether the regex `"b"` will match the string `"abc"`. Setting this to `false` with `"optional":true` can be used as a "search and replace" mode.

#### Field: `multiline_handling`
**Optional** string, defaults to `"first_match_only"`

How the regex will behave when fed a string that contains multiple lines:
- **`regex_default`**: The default behaviour of regex (but not of this transform), which may be unintuitive to newcomers.
- **`first_match_only`**: Each line is separately fed into the regex until one matches. Only the substitution for this line is returned. The remaining lines are ignored.


## Transform: `test`
**Accepted input:** Any

Checks whether another transform fails or succeds. On success, the original, unmodified input is returned, instead of the inner transform's output.
### Schema
```json
{
	"function": "test",
	"tester": [ 
		"lowercase",
		{ "whitelist": [ "..." ] }
	]
}
```
In this example, the whitelist will evaluate a lowercase string, but the capitalized string will be returned on success.

## Transform: `alternative`
**Accepted input:** Any

Contains multiple chains of transforms, and tests them independently until *one of them* succeeds. This returns the result of the first transform to succeed. This fails if all inner transforms fail.

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
**Accepted input:** String

Fails or succeeds depending on whether the input is in the given list.
The original value is returned on success.

This 
###	Schema
```json
{
	"whitelist": [
		"allowed_value_1",
		"allowed_value_2",
		"allowed_value_3",
	]
}
```
```json
{
	"blacklist": [
		"disallowed_value_1",
		"disallowed_value_2",
		"disallowed_value_3",
	]
}
```
> Specifying the `function` type is optional for these transforms.


## Transform: `remap`
**Accepted input:** String

If the input is amongst the given keys, this returns the associated value.
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
**Accepted input:** String

Individually replaces or delete characters in the input string.
This can, for example, be used prior to `sanitize`, in order to preserve meaningful characters that would otherwise be removed.

This transform never fails.

### Schema
```json
{
	"function": "charset_remap",
	"source":      "абвгдезийклмнопрстуфхюя",
	"destination": "abvgdezijklmnoprstufxyy",
	"delete": "ь",
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

## Transform: `equals`
**Accepted input:** String or Number

Fails if its input is not strictly equals to the given value. Returns its input on success.

### Schema:
```json
{
	"equals": "value"
}
```
```json
{
	"equals": 5
}
```
> Specifying `"function":"equals"` is optional for this transform.

## Transform: `smaller_than`, `smaller_or_equals`, `greater_than`, `greater_or_equals`
**Accepted input:** Number

Fails if its input does not appropriately compare to the given value. Returns its input on success.
### Schema:
```json
{
	"smaller_than": 5
}
```
> Specifying the `function` type is optional for these transforms.
