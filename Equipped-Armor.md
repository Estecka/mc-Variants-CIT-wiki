# Equippable Modules

Changing the look of equipped armor works similarly to changing inventory textures; this page here will only cover the differences. If you are not aready familiar with Variants-CIT, makre sure to follow the [introductory turtorial](./Getting%20Started) first.

The fundamental difference is that equippable modules will override the `assetId` of the ['`equippable`'](https://minecraft.wiki/w/Data_component_format#equippable) component, instead of the `item_model` component previously.

Modules are able to change the look of equipment on any mob that makes use of the `equippable` component, not just humanoids. (See the [History](https://minecraft.wiki/w/Equipment#History) table on the equipment assets wiki.)

Only the textures of equipments can be changed that way, not their 3D model.

## In short:
Set the [`hook`](./Module-Configuration#field-hook) of your module to `"equippable"`. For humanoid armors, set its [`assetGen`](./Module-Configuration#field-assetgen) to `"equipment/humanoid"`.
```json
{
	"hook": "equippable",
	"type": "...",
	"items": "...",
	"modelPrefix": "...",
	"assetGen": "equipment/humanoid"
}
```

It's technically possible for a module to pull double-duty, and change both the equipped and inventory look of an item. However, because of the discrepancy in how `item_model` assets and `equippable` assets are structured, creating two separate modules will be more convenient in most cases.


## Asset Types
Instead of collecting Item States, Baked Models and Textures, equippable modules will collect ['`equipment`' assets](https://minecraft.wiki/w/Equipment).

Variant ID                                  | `<namespace>:<path>`
------------------------------------------- | :-------------------
Equivalent `equippable` component (assetId) | `<namespace>:<modelPrefix><path>`
Matching equipment model                    | `/assets/<namespace>/equipment/<modelPrefix><path>.json`
Matching texture                            | `/assets/<namespace>/textures/entity/equipment/<layer>/<modelPrefix><path>.png`

Equipements models can have multiple layers, meaning you only need a single model (and thus a single module) to handle all the body parts of an armor tier, potentially even across multiple mobs !

## Asset Generation
For equipments, there are only a couple of [generator presets](./Asset-Generation#built-in-asset-generator-presets) which should cover the most common use cases: `equipment/humanoid` (full-body armor) and `equipment/elytra`.

The existence of a single layer's texture is enough to trigger the creation of a model, but generators can only create json's with a fixed structure. As a result, generated models will always have *all* the layers defined by the template, even if some of the corresponding textures were not provided.
