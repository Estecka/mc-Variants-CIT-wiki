# Custom Module Types

Custom CIT modules may be defined by other mods, for use in [module configurations](Module-Configuration).

A module's primary function is to figure out the variant of an item, from which the mod will derive the model to use. Optionally, it can implement additional logic and special models for situations that are not covered by the variant system.

The example codes below use **Yarn Mappings**.

## The simplest module
The smallest possible module is a simple function, that takes in an `ItemStack`, and returns the `Identifier` for its variant. This may return null if no variant was found, which will hand over control to another module.

You may register either a lambda, or an instance of `ISimpleCitModule`, alongside an ID which will be used as the `type` in resource packs. 

```java
public class CustomModule
implements ISimpleCitModule
{
	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), new CustomModule());
	}

	@Override
	public @Nullable Identifier GetItemVariant(ItemStack stack){
		return stack.get(someComponent).GetVariant();
	}
}
```

## Special Modules
Modules with special models must implements the super-type `ICitModule`; they are registered the same way as Simple modules.

Your logic will instead be implemented in `GetItemModel`. Instead of returning a variant ID, it directly returns the ID of the model to use (or null).
You can access the list of model IDs through the `IVariantManager` passed as parameter; special models use the same keys that are defined in the `special` block of the module configuration.


```java
public class CustomModule
extends ICitModule
{
	@Override
	public @Nullable Identifier GetItemModel(ItemStack stack, IVariantManager variantManager){
		if (/*Special condition*/)
			return variantManager.GetSpecialModel("specialKey");
		else
			return variantManager.GetVariantModel(variantId);
	}
}
```

## Custom Parameters
Modules that receive custom parameters must be instantiated per-config. Instead of directly registering an instance of it, you must register a **codec** which will be applied to the the `parameters` block in the module configuration.

```java
public class CustomModule
extends ICitModule, ISimpleModule //either work
{
	static public final MapCodec<CustomModule> CODEC = RecordCodecBuilder.mapCodec(instance->instance
		.group(
			Codec.BOOL.fieldOf("param1").forGetter(/*...*/),
			Codec.INT.fieldOf("param2").forGetter(/*...*/)
		)
		.apply(instance, CustomModule::new)
	);

	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), CODEC);
	}

	public CustomModule(boolean param1, int param2){
		//...
	}

	// Main logic
	// ...
}
```
