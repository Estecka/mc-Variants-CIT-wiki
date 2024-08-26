# Custom Module Types

Custom CIT modules may be defined by mods, for use in [module configurations](Module-Configuration).

A module's primary function is to figure out the variant of an item, from which the mod will derive the model to use. Optionally, it can implement additional logic and special models for situations that are not covered by the mod.

The example codes below use **Yarn Mappings**.

## The simplest module
The smallest possible module is a simple function, that takes in an `ItemStack`, and returns the `Identifier` for its variant. This may return null if no variant was found, which will use the vanilla model.

You then register an instance of this module with an ID, which will be used as the `type` in resource packs. You may register either a lambda, or an instance of `ISimpleCitModule`.

```java
public class CustomModule
implements ISimpleCitModule
{
	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), new CustomModule());
	}

	@Override
	public @Nullable Identifier GetItemVariant(ItemStack stack){
		// Base logic
	}
}
```

## Special Modules
Modules with special models must implements the more complete interface `ICitModule`.
Unlike the simpler modules, those need to be instantiated per configuration; so instead of directly registering an instance of it, you must register a constructor or equivalent. That constructor accepts the list of special models that were configured by the resource pack.

Your special logic must be implemented in `GetItemModel`, from which you may return any of the models provided via the constructor.
There is no need to evaluate `GetItemVariant` in there; if no special condition apply, you should return the value from the provided variant manager. Returning `null` will again cause the vanilla model to be used instead.

```java
public class CustomModule
extends ICitModule
{
	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), CustomModule::new);
	}

	private final @Nullable ModelIdentifier mySpecialModel;

	public CustomModel(Map<String,ModelIdentifier> specialModels){
		this.mySpecialModel = specialModels.get("myKey");
	}

	@Override
	public @Nullable Identifier GetItemVariant(ItemStack stack){
		// Base logic
	}

	@Override
	public @Nullable ModelIdentifier GetItemModel(ItemStack stack, IVariantManager variantManager){
		if (/*Special logic*/)
			return mySpecialModel;
		else
			return variantManager.GetVariantModel(stack);
	}
}
```

## Custom Parameters
Modules that wish to receive custom parameters must be registered as a constructor, alongside a codec which will be used to decode the `parameter` object in the module configuration.

Those modules can implement either `ISimpleCitModule` or `ICitModule`. The parameters will be passed to the modules constructor, after the special models list if applicable.

```java
public class CustomModule
extends ISimpleCitModule
{
	static public final MapCodec<MyDataType> CODEC = /*...*/;

	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), CustomModule::new, CODEC);
	}

	private final MyDataType parameters

	public CustomModel(MyDataType parameters){
		this.parameters = parameters;
	}

	// ...
}
```
or
```java
public class CustomModule
extends ICitModule
{
	static public final MapCodec<MyDataType> CODEC = /*...*/;

	static public void Register(){
		ModuleRegistrar.Register(Identifier.of("modid","custom_module"), CustomModule::new, CODEC);
	}

	private final MyDataType parameters
	private final @Nullable ModelIdentifier mySpecialModel;

	public CustomModel(Map<String,ModelIdentifier> specialModels, MyDataType parameters){
		this.parameters = parameters;
		this.mySpecialModel = specialModels.get("myKey");
	}

	// ...
}
```
