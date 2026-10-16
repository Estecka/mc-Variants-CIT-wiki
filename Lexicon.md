# Lexicon

> [!NOTE]
> 
> This page is a **Work in Progress**

- **Asset Generator**
- [**Baked Models**](https://minecraft.wiki/w/Model)	=> `assets/*/models/item/**/*.json`  
  Although Minecraft also supports baked models in other subdirectories of `models/`, Variants-CIT modules can only collect the ones in `models/item/`
- **Equippable**
- **Equipment** : TBD. Either '*Equipment Model*', or '*Items Stacks*' with an `equippable` component.
- **Equipment Definition**	=> *Equipment Model*
- **Equipment Layer**	
- **Equipment Model**	=> `assets/*/equipments/**/*.json`
- **Equipment Texture**	=> `assets/*/textures/entity/equipment/*/**/*.png`
- **Intrinsic Models** : Internal term for models that were defined in the *Parameters* of the `predicates` Module type, instead of using `modelList` or `modelPrefix`.
- **Item** : Either '*Item Type*' or '*Item Stack*'. Usually '*Item Stack*'
- **Item Definition**	=> *Item States*
- **Item Model** :	Ambiguous; *do not use*. Use **Item States** or **Baked Model**, depending on what you are refering to.
- **Item Model Definition**	=> *Item States*
- **Item Stack** : An actual instance of an item. The stuff in your inventory.
- [**Item States**](https://minecraft.wiki/w/Items_model_definition)	=> `assets/*/items/**/*.json`
- **Item Texture**	=> `assets/*/textures/item/**/*.json`
- **Item Type**
- **Model** :	Any asset that a module can collect. Could refer to either '*Item States*', '*Equipment Models*', '*Baked Models*', or even '*Textures*'.
Usage is acceptable only as an umbrella term when the exact kind of model does not matter, or as a pronoun when there's no possible doubt what it's referring to.
- **Model ID** :	Should specifically refers to Model ID used in Modules. Not the IDs used in Vanilla.
- **Module** => *Module Definition*
- **Module Configuration** => *Module Definition*, Use should be deprecated in favor of the later.
- [**Module Definition**](./Module-Configuration)	=> `assets/*/variants-cit/modules/**/*.json`
- [**Module Type**](./Module-Types) : The logical unit of a module. The part that looks at an '*item stack*', and computes its '*Variant ID*' or '*Model ID*'. Often does so independently of available models.
- [**Precondition**](./Precondition%20Cheat-Sheet)	
- [**Parameters**](./Precondition%20Cheat-Sheet) : The `parameters` block inside a *Module Definition*. Avoid any other use of this term.
- **Texture**	=> `assets/*/textures/**/*.png`
- **Transform** :	Either a '*Transform Type*', a '*Transform Chain*', or an individual transform within a chain.
- [**Transform Chain**](./Transforms)	
- **Transform Chain Array**	
- **Transform Type**	
- **Variant** : A possible variation of an '*item stack*'.
- **Variant ID** : The identifier that represents a '*Variant*', independently of models.
