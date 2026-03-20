# Variants-CIT

Variants-CIT is a mod for managing large collection of item variants that all follow the same rule, without needing to define each variant individually: Just point at the folder containing your models, and the mod will automatically associate each model to the item variant with a matching name.

- [FAQ](./FAQ)
- [Getting Started](./Getting%20Started)
- [Troubleshooting](./Troubbleshooting)


## Definitions & Disambiguation
### Vanilla concepts:

- **"[Item Model Definition](https://minecraft.wiki/w/Items_model_definition)"** or **"Item states"** refers to the assets under the "`items/`" folder of a resource pack. Throughout this wiki, "Item States" will be preferred to avoid confusion with other kinds of "models".  
  Since MC 1.21.4, item States are the fundamental asset type used in vanilla resource packs to change the appearance of an item stack. The item component **`item_model`** refers to one of these assets, and Variants-CIT works by simply overriding this component.
- **"[Models](https://minecraft.wiki/w/Model)"** or **"Baked Model"** refers to the assets under the "`models/`" folder. Throughout this wiki, "Baked Model" will be preferred to avoid confusion with other kinds of "models".  
  In older versions of Minecraft, baked models were the fundamental asset type used to change an item stack's model, but are now controlled through item states.
- **"[Equipment](https://minecraft.wiki/w/Equipment)"** or **"Equippable"** refers to the assets under the `equipment/` folder of a resource pack.  
  Equipments are the fundamental asset type used to control the texture of an equipped piece of armor. The item component **`equippable`** refers to one of these assets, and Variant-CIT works by simply overriding this component.

### Variants-CIT concepts:
- **"[Module](./Module-Configuration)"** refers to the assets under the `variants-cit/modules/` folder of resource packs. A module's job is to figure out the variant ID of an item stack, and give it the corresponding model or texture from a large collection. 
  In most cases, a single module is enough to manage all the variants of an item.
- **"Variant ID"** or simply **"Variant"**, is an intermediary piece of data that a module will calculate for an item stack, right before choosing its model or texture.  
  In broad terms, the variant is the data you are looking for in an item: for an enchanted book module, the variant will be the enchantment. For a custom name module, the variant will be the item's name.  
  Sometimes the variant is just a verbatim copy of the value found on the item. Othertimes, the variant can be an agglomerate of multiple pieces of data (e.g: enchantment id + enchantment level), or be a sanitized version of the raw data (e.g: removing illegal characters from a custom name). This depends on the module type and how it was configured.  
  An item's model ID depends directly from its variant ID. Models and variant have a one-to-one relationship in both direction.  
Item stack -> raw data -> variant-id -> model-id
- **"Variant Asset"** or **"Variant Model"** is the vague concept of any asset that a module can collect or associate to a variant. The fundamental variant assets types are **Item States** and **Equipment Models**, but with the right asset-generation options, a module can also collect **textures** and **baked models**, and will automatically create the underlying assets.

## External Resources
Variants-CIT relies on the vanilla format for item models. Here are some relevant pages from the minecraft wiki:
- [Equipment](https://minecraft.wiki/w/Equipment)
- [Identifier](https://minecraft.wiki/w/Identifier)
- [Item model definition](https://minecraft.wiki/w/Items_model_definition)
- [Model](https://minecraft.wiki/w/Model)
- [Tutorial: Creating a resource pack](https://minecraft.wiki/w/Tutorial:Creating_a_resource_pack)
- [Tutorial: Models](https://minecraft.wiki/w/Tutorial:Models#Item_models)