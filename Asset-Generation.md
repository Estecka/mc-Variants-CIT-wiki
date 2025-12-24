# Asset Generation (Alpha)

> [!WARNING]
> 
> **This feature is in alpha, and may receive breaking changes.**

Asset Generation is a more powerful alternative to the old-school `modelParent` option. It can generate item states, baked models and equipment models, with arbitrary JSON content.

This page only describes the syntax of individual generators, and their resource types. See [Module Configuration](./Module-Configuration#field-assetgen), for how to bind a generator to a module.


# Built-In Asset Generator Presets
For the layman, presets are the easy way to change an item's texture by providing only textures. Just set your module's `assetGen` to one of the values below.

```json
{
	"type": "...",
	"modelPrefix": "...",
	"assetGen": "item_model/generated"
}
```

### Preset: `item_model/generated`
Creates basic models for a simple item with no animation. This is equivalent to the old-school option `"modelParent":"item/generated"`.

### Preset: `item_model/handheld`
Creates basic models for a simple tool with no animation. This is equivalent to the old-school option `"modelParent":"item/handheld"`.

### Preset: `item_model/bow`
Creates models and items states with the same structure as the vanilla bow.
Each variant should also provide textures or baked models with the following suffixes:
- `_pulling_0`
- `_pulling_1`
- `_pulling_2`

### Preset: `item_model/crossbow`
Creates models and items states with the same structure as the vanilla crossbow.
Each variant should also provide textures or baked models with the following suffixes:
- `_arrow`
- `_rocket`
- `_pulling_0`
- `_pulling_1`
- `_pulling_2`

### Preset: `item_model/fishing_rod`
Creates models and items states with the same structure as the vanilla fishing rod.
Each variant should also provide textures or baked models with the following suffixes:
- `_cast`

### Preset: `item_model/goat_horn`
Creates models and items states similar to the vanilla goat horn.
No additional textures need be provided for the tooting animation.

Custom tooting models can be provided with a `_tooting` suffix. Do note that this a **SUFFIX**, unlike the vanilla goat horn which uses `tooting_` as a *prefix*.

### Preset: `item_model/trident`
Creates models and items states similar to the vanilla trident.

Each variant should also provide a texture suffixed with:
- `_in_hand`

_or_ baked models suffixed with:
- `_in_hand`
- `_throwing`

The auto-generated `_throwing` model will use the `_in_hand` texture.

Generated models will have the same shape as the vanilla trident, and expect a similarly formatted texture.
Unlike vanilla, **the `_in_hand` texture must be located in the same folder as the GUI textures**.

### Preset: `item_model/trident_gui_only`
Creates models for a trident, but only replace the inventory texture. The in-hand model will use the vanilla trident model and texture.

### Preset: `equipment/humanoid`
Creates two-layers equipment models for humanoid armors.
You should provide textures for the following layers:
- `humanoid/`
- `humanoid_leggings/`

### Preset: `equipment/elytra`
Creates single-layer equipment models for an elytra. The elytra will use the player's cape texture if any.
You should provide textures for the following layers:
- `wings/`

### Preset: `equipment/saddle`
Creates equipment models for a saddle which supports all animals.
You should provide textures for the following layers:
- `camel_saddle/`
- `donkey_saddle/`
- `horse_saddle/`
- `mule_saddle/`
- `pig_saddle/`
- `skeleton_horse_saddle/`
- `strider_saddle/`
- `zombie_horse_saddle/`

# Custom Asset Generators
## Pipeline
The process of generating assets is similar to that of the old-school `modelParent` option :

1. A module collects all textures, models, etc, that matches its model prefix and special models. It then pours those assets into the asset generators.
2. The IDs of those assets are tested against a generator's input regex, to check whether the generator is allowed to create a new asset.
3. The generators uses the input asset's ID to create a bunch of variables, which are used to fill a template, and compose the ID of the output asset.
4. The generated assets are added to a virtual resource pack. This pack is located at the bottom of the stack, so generators can never overwrite already existing assets.
5. The module creates its library of variants, collecting both the pre-existing and generated assets.

If multiple generators within a module want to generate the same asset, the firstmost generator in the array will take priority. If multiple modules want to generate the same asset, the module with the most specific `modelPrefix` will take priority.

You can use the command `/variants-cit assetGen` to quickly check the content of any generated asset.
 
## Templates
In short: Templates are just big substitution strings.

Templates are a resource type used as the foundation for every generated assets. They can look like assets of any other type, but have some data missing in them, which will be filled in by the asset generators.

To serve as examples, you can find all built-in templates here:
[assets/variants-cit/variants-cit/templates/](https://github.com/Estecka/mc-Variants-CIT/tree/HEAD/src/main/resources/assets/variants-cit/variants-cit/templates)

Custom templates can be provided under `variants-cit/templates/`. For now only JSON files are gathered, but this may be extended to other file extensions in the future.

Variable `${names}` can contain any character in `[a-zA-Z0-9_]`, but can never start with a number. They must always be surrounded by curly braces.

Some variable will be automatically generated by the mod, and be available to all templates. By convention, those variable names will always use `SCREAMING_SNAKE_CASE`. Some built-in templates still contain variables in `camelCase`, which must be defined in every generator using that template.

### Auto-generated variables:
- **`${NAMESPACE}`** :  
  The namespace being worked with. All IDs mentionned below always have the same namespace.

- **`${INPUT_PATH}`**, **`${INPUT_ID}`** :  
  The path and full identifier of the asset that triggered the generator.

- **`${OUTPUT_PATH}`**, **`${OUTPUT_ID}`** :  
  The path and full identifier of the asset being generated.

- **`${RADICAL_PATH}`**, **`${RADICAL_ID}`** :  
  The path and full identifier of the radical. (See [Radical](#field-radicalpath))

## Custom Presets
Generator presets can be added in `variants-cit/assetgen_presets/`, and then referenced from multiple modules. Generators stored here won't do anything until they are referenced from at least one module.

Creating a preset is not a requirement; generators can be defined directly inside a module.

Generator presets must always be declared inside an array and cannot contain references to other presets.


## Schema
The smallest possible generator only needs to define the asset types it works with, and the template to use :
```json
{
	"pass": "models_from_textures",
	"template": "models/item/generated"
}
```
Below is a more complete generator. This extract from the `item_model/trident` preset looks for `_in_hand` textures, and generate `_throwing` baked models using that texture :
```json
{
	"pass": "models_from_textures",

	"inputPath": "(.*)_in_hand",
	"outputPath": "$1_throwing",
	"radicalPath": "$1",

	"template": "models/parent",
	"variables": {
		"modelParent": "variants-cit:item/trident_throwing"
	}
}
```

For more examples, you can find the other built-in presets here:
[assets/variants-cit/variants-cit/assetgen_presets/](https://github.com/Estecka/mc-Variants-CIT/tree/HEAD/src/main/resources/assets/variants-cit/variants-cit/assetgen_presets)

### Field: `pass`
**Mandatory**, String.

Indicates which asset type is accepted as input, and which asset type is produced.
This also affects which portion of the asset ID will be tested against the input regex, and how the output path will be interpreted:

Pass                           | Input Files                                 | Output Files                    | Radical
------------------------------ | ------------------------------------------- | ------------------------------- | -------
**`equipments_from_textures`** | textures/entity/equipment/`<inputPath>`.png | equipment/`<outputPath>`.json   | equipment/`<radicalPath>`.json
**`models_from_textures`**     | textures/item/`<inputPath>`.png             | models/item/`<outputPath>`.json | items/`<radicalPath>`.json
**`items_from_models`**        | models/item/`<inputPath>`.json              | items/`<outputPath>`.json       | items/`<radicalPath>`.json

Any model generated in the `models_from_texture` pass will then be received by the `items_from_models` pass.

The input path for equipment textures includes the layer name, wich you will usually want to exclude from the radical and output path.

### Field: `inputPath`
**Optional** Regex, defaults to `".*"` (accept everything).

Describes what assets can trigger this generator.
This regex is matched against a portion of input asset's path (See [Pass](#field-pass)). If the input matches, the defined output is created.

A generator will only ever receive the assets that were collected by its module. I.e: all assets matching the `modelPrefix`, the `fallback` model, and the `special` models. There is no need to write a regex specifically to exclude other assets. (See also: [Radical](#field-radicalpath))


### Field: `outputPath`
**Optional** String, defaults to `"$0"` (same as input).

The path of the identifier of the asset that will be generated. This is a regex substitution string, which can use values captured by the input regex.
This only represent a portion of the output file's path. (See: [Pass](#field-pass))

### Field: `radicalPath`
**Optional** String, defaults to `"$0"` (same as input).

Indicate the ID of the core/master asset that controls the input asset. I.e, the item states or equipment model that controls a texture or a baked model.
A generator's output will be discarded if the output's radical cannot be collected by the module.

This allows the generator to be triggered by assets that would normally not be collected by a module.
This is is particularly relevant to "subvariant" assets, such as `_pulling` for bows, `humanoid_leggins/` for armor, etc.

Some scenarios where having a correct radical is important are:
- **You are generating assets for a special or a fallback model.**  
Eg: With `"fallback":"xyz"`, the generator would normally receive the texture called `xyz.png`, but not `xyz_pulling.png`.
- **You are using a module type with a very syntax on model names.**  
Eg: The `durability` module would normally not collect a texture called `50_pulling.png`, even if it matches the model prefix because `_pulling` is not a valid number.
- **You are generating equipment assets.**  
All input textures include the layer name at the start of their id (Eg: `humanoid/modelPrefix/tex.png`), and so will fail to match the modelPrefix, or any fallback/special models.

> [!WARNING]
> 
> The name of this field is subject to changes. Suggestions are welcome.

### Field: `template`
**Mandatory**, namespaced identifier.

The identifier of the [template](#templates) used to generate an asset. This identifier defaults to the `variants-cit` namespace, instead of `minecraft`.

### Field: `variables`
**Optional**, maps variable names to their values.

Hardcoded variables that will be used to fill the template. Any variable defined here will override auto-generated variables.
