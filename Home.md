# Variants-CIT

Variants-CIT is a mod for managing large collection of item variants that all follow the same rule, without needing to define each variant individually: Just point at the folder containing your models, and the mod will automatically associate each model to the item variant with a matching name.

**For resource pack makers:**
- [FAQ](./FAQ)
- [Getting Started & Troubleshooting](./Getting%20Started%20&%20Troubleshooting)
- [Module Types](./Module-Types)
- [Configuring CIT Modules](./Module-Configuration)

**For mod developpers:**
- [Creating custom module types](./Java-API)


## Definitions & Disambiguation
### Vanilla concepts:

- **"Item state"** refers to the assets under the "`items/`" folder of a resource pack.
- **"Baked Model"** or simply **"model"** refers to the assets under the "`models/`" folder, and only those assets.
- **"Model selector"** refers to the various "model" fields found inside of item states.

"Item model" shall not be used, because of its ambiguity between the "`items/`" and "`models/`" folders.

### Variants-CIT concepts:
- **"Module"** refers to the assets under the `variants-cit/item/` folder of resource packs. A module's job is to figure out the variant ID of an item stack, and give it the corresponding model or texture from a large collection.
- **"Variant ID"** or simply **"variant"**, is an intermediary piece of data that a module will calculate for an item stack, right before choosing its model or texture.  
In broad terms, the variant is the data you are looking for in an item: for an enchanted book module, the variant will be the enchantment, for a custom name module, the variant will be the item's name. In many cases, the variant is just a verbatim copy of the value found on the item. Othertimes, the variant can be an agglomerate of multiple pieces of data (e.g: enchantment id + enchantment level), or be a sanitized version of the raw data (e.g: removing illegal characters from a custom name).  
The variant id is directly used to construct the item's model/texture id. Model and variant ids have a one-to-one relationship in both direction.  
Item stack -> raw data -> variant-id -> model-id
- **"CIT"** refers to the vague concept of any asset that a module can associate to a variant. A CIT can be either an **item state**, a **baked model**, or a **texture**. This word will only be used in contexts where the distinction does not matter.

## External Resources
Variants-CIT relies on the vanilla format for item models. Here are some relevant pages from the minecraft wiki:
- [Resource Location](https://minecraft.wiki/w/Resource_location) (a.k.a. Identifiers)
- [Tutorial:Creating a resource pack](https://minecraft.wiki/w/Tutorial:Creating_a_resource_pack)
- [Tutorial:Models](https://minecraft.wiki/w/Tutorial:Models#Item_models)
- [Model](https://minecraft.wiki/w/Model) (a.k.a. Baked models)
- [Item model definition](https://minecraft.wiki/w/Items_model_definition) (a.k.a. Item states and Model selectors)
