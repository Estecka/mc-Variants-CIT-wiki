# Equippable Modules

> [!IMPORTANT]
>
> This feature is exclusive to VCIT v3 and v4. (MC 1.21.4 and onward)

Changing the look of equipped armor works similarly regular items, so most of what is written in the rest of the wiki still applies here. This page here will only cover the differences.

The main difference is that equippable modules will override the ['`equippable`'](https://minecraft.wiki/w/Data_component_format#equippable) component's `assetId` field, instead of the `item_model` component previously.

Modules should be able to change the look of equipment on any mob, not just humanoids, so long as this is supported by your version of Minecraft. (See the [History](https://minecraft.wiki/w/Equipment#History) table on the equipment assets wiki.)

Only the textures of equipments can be changed that way, not their 3D model.

## Enabling the feature
Add a new `context` field to your module:
```json
{
	"context": "equippable",
	"type": "...",
	"items": [ "..." ],
	"modelPrefix": "..."
}
```

### `context`

**Optional**, a single, or an array of strings. Defaults to `"item_model"`.

Possible values are:
- **`item_model`**: causes the module to override the `item_model` component of the item. (Normal behaviour.)
- **`equippable`**: causes the module to override the `equippable` component of the item, or more specifically, the '`assetId`' field inside that component.

It's technically possible for a module to pull double-duty, and change both aspects of an item at once. However, it's unlikely to find concrete uses, because of the discrepancy in how item assets and equipment assets are structured. Feedback on that matter would be appreciated.


## Asset Types
Instead of collecting Item States, Baked Models and Textures, equippable modules will collect ['`equipment`' assets](https://minecraft.wiki/w/Equipment).

Variant ID                                  | `<namespace>:<path>`
------------------------------------------- | :-------------------
Equivalent `equippable` component (assetId) | `<namespace>:<modelPrefix><path>`
Matching equipment asset                    | `/assets/<namespace>/equipment/<modelPrefix><path>.json`

**Equippable modules can only work with this type of assets.** Those assets cannot be automatically generated from textures at this time.
On the flip-side, you only need a single equipment assets (and thus a single module) to handle all the body parts of an armor tier.

The fields `modelParent` and `itemsFromModels` have no effect on the `"equippable"` context of a module.

Depending on the layer type, the textures associated with an equipement are:  
`/assets/<namespace>/textures/entity/equipment/<layer_type>/<path>.png`


