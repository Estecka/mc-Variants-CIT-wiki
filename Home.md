# Variants-CIT

Variants-CIT is a mod for managing large collection of item variants that all follow the same rule, without needing to define each variant individually: Just point at the folder containing your models, and the mod will automatically associate each model to the item variant with a matching name.

- [FAQ](./FAQ)
- [Getting Started](./Getting-Started)
- [Troubleshooting](./Troubbleshooting)


## Definitions & Disambiguation
### Vanilla concepts:

- **"[Item Model Definition](https://minecraft.wiki/w/Items_model_definition)"** or **"Item states"** refers to the assets under the "`items/`" folder of a resource pack. Throughout this wiki, **"Item States"** will be preferred to avoid confusion with other kinds of "models".  
  Since MC 1.21.4, item States are the fundamental asset type used in vanilla resource packs to change the appearance of an item stack. The item component **`item_model`** refers to one of these assets, and Variants-CIT works by simply overriding this component.
- **"[Model](https://minecraft.wiki/w/Model)"** or **"Baked Model"** refers to the assets under the "`models/`" folder. Throughout this wiki, **"Baked Model"** will be preferred to avoid confusion with other kinds of "models".  
  In older versions of Minecraft, baked models were the fundamental asset type used to change an item stack's model, but are now controlled through item states.
- **"[Equipment Definition](https://minecraft.wiki/w/Equipment)"** or **"Equipment Model"** refers to the assets under the `equipment/` folder of a resource pack.  
  Equipments are the fundamental asset type used to control the texture of an equipped piece of armor. The item component **`equippable`** refers to one of these assets, and Variant-CIT works by simply overriding this component.

### Variants-CIT concepts:
- **"[Module](./Module-Configuration)"** refers to the assets under the `variants-cit/modules/` folder of resource packs. A module's job is to figure out the variant ID of an item stack, and give it the corresponding model or texture from a large collection. 
  In most cases, a single module is enough to manage all the variants of an item.
- **"Model ID"** is the final piece of data that a module computes for an item stack. Model IDs can refer to different asset types depending onthe the module's hook. See: [What file for a model ID?](./Understanding-AssetGen#what-files-corresponds-to-a-model-id)
- **"Variant ID"**. For modules that use a `modelPrefix`, the variant ID is an intermediary piece of data that is computed for an item stack, right before picking its Model ID.  
  In broad terms, the variant is the data you are looking for in an item: for an enchanted book module, the variant will be the enchantment. For a custom name module, the variant will be the item's name.  
  Sometimes the variant is just a verbatim copy of the value found on the item. Othertimes, it is an agglomerate of multiple pieces of data (e.g: enchantment id + enchantment level), or be a sanitized version of the raw data (e.g: removing illegal characters from a custom name).

## External Resources
Variants-CIT relies on the vanilla format for item models. Here are some relevant pages from the minecraft wiki:
- [Equipment](https://minecraft.wiki/w/Equipment)
- [Identifier](https://minecraft.wiki/w/Identifier)
- [Item model definition](https://minecraft.wiki/w/Items_model_definition)
- [Model](https://minecraft.wiki/w/Model)
- [Tutorial: Creating a resource pack](https://minecraft.wiki/w/Tutorial:Creating_a_resource_pack)
- [Tutorial: Models](https://minecraft.wiki/w/Tutorial:Models#Item_models)