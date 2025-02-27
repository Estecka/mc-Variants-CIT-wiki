# NBT Adapter

NBT Adatpers are a type of parameter used to extract data from an NBT-encoded component. They can perform some basic validation and transformation on the data, and return a string that is then used in the construction of the item's variant ID.

## Schema:

```json
{
	"componentType": "fireworks",
	"nbtPath": ".array[0].map{0}.key",
	"expect": "rich_text",
	"transform": "sanitize"
}
```

### `componentType`
**Mandatory,** Namespaced identifier

The identifier of the component to explore.

### `nbtPath`
**Mandatory,** String

The location of the data to extract within the input NBT. The path can be an emptry string, in cases when the data is the input NBT itself.

- `.keyname` Expects a compound, and returns the value under the given name. "keyname" can contain any character that is legal for a namespaced identifier to hold, including columns `':'`, but excluding dots `'.'`.

- `[n]` expects an array, and gets the value at the given index. "n" must be a number.

- `{n}` expects a compound, but accesses it like an array of key-value pairs, and returns the pair at the given index. It should always be followed by either `.key` or `.value`.
The order in which entries are sorted is undefined.

### `expect`
**Optional,** String, defaults to `"primitive"`.

The expected data type at the end of the path, and how to interpret it.
All extracted data are converted to string, and then fed into the transforms.
Invalid data will be treated the same as if the data were missing.


Possible values are:
- `string`: Expects any string and extracts it as-is.
- `number`: Expects any number type. The type suffix is absent from the extracted string. (E.g: `1b` in the NBT will be interpreted as simply `1`)
- `primitive`: Expects either a string or a number.
- `identifier`: Expects a valid namespaced identifier. If the data has no namespace, `minecraft:` is made explicit in the extracted string.
- `rich_text`: Expects a stringified rich text, and extracts a plain text version, stripped of all formatting.

### `transform`
**Optional,** a single String, or an array of Strings.

A series of transformations applied to the previously extracted string. The order of the items in the array matters.

Possible values are:
- `lowercase`: Converts all upper-case to lower-case.
- `sanitize`: Removes all characters that are illegal for an identifier. Uppercases are replaced with lowercases, spaces are replaced with underscores, accentuated characters have their accents stripped, and all other invalid characters are removed completely.
**The result is not guaranteed to be a valid identifier**; for example it may still contain more than one column `':'`.
- `sanitize_path`: Like `sanitize`, but also strips all characters that are illegal for an identifier's path.
- `sanitize_namespace`: Like `sanitize`, but also strips all characters that are illegal for an identifier's namespace.
- `discard_namespace`: Removes `':'` and all preceding characters. Behaviour is undefined on strings that contain multiple columns.
- `discard_path`: Removes `':'` and all following characters. Behaviour is undefined on strings that contain multiple columns.
