# Equippable Modules

> [!IMPORTANT]
>
> This feature is exclusive to VCIT v3 and v4. (MC 1.21.4 and onward)

Changing the look of equipped armor works similarly to changing inventory textures; this page here will only cover the differences.

The main difference is that equippable modules will override the ['`equippable`'](https://minecraft.wiki/w/Data_component_format#equippable) component's `assetId` field, instead of the `item_model` component previously.

Modules should be able to change the look of equipment on any mob, not just humanoids, so long as this is supported by your version of Minecraft. (See the [History](https://minecraft.wiki/w/Equipment#History) table on the equipment assets wiki.)

Only the textures of equipments can be changed that way, not their 3D model.

## Enabling the feature
Set the [`context`](./Module-Configuration#field-context) of your module to `equippable` :
```json
{
	"context": "equippable",
	"type": "...",
	"items": "...",
	"modelPrefix": "..."
}
```

It's technically possible for a module to pull double-duty, and change both aspects of an item at once. However, it's unlikely to find concrete uses, because of the discrepancy in how item assets and equipment assets are structured.


## Asset Types
Instead of collecting Item States, Baked Models and Textures, equippable modules will collect ['`equipment`' assets](https://minecraft.wiki/w/Equipment).

Variant ID                                  | `<namespace>:<path>`
------------------------------------------- | :-------------------
Equivalent `equippable` component (assetId) | `<namespace>:<modelPrefix><path>`
Matching equipment model                    | `/assets/<namespace>/equipment/<modelPrefix><path>.json`
Matching texture                            | `/assets/<namespace>/entity/equipment/<layer>/<modelPrefix><path>.json`

Equipements can work with multiple layers, meaning you only need a single model (and thus a single module) to handle all the body parts of an armor tier !

## Asset Generation
Using [`assetGen`](./Module-Configuration#field-assetgen), equipment models can automatically be generated from textures.

For equipments, there are only a couple of [generator presets](./Asset-Generation#built-in-asset-generator-presets) which should cover the most common use cases: `equipment/humanoid` (full-body armor) and `equipment/elytra`.

The existence of a single layer's texture is enough to trigger the creation of a model, but generators can only create json's with a fixed structure. As a result, generated models will always have *all* the layers defined by the template, even if some of the corresponding textures were not provided.
