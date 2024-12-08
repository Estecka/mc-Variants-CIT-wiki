# Override Predicates

Override predicates are the vanilla way of defining variants of a model, based on numerical data. Those can be used inside the identifier-based models managed by Variant-CIT's, in order to provide a second layer of variance to the item.

> [!IMPORTANT]
> 
> This page is only applicable to MC 1.21.3 and older.

## `bucket_entity_age`
**Applicable items:** all.

Returns the value of `Age` in the `bucket_entity_data` component, or `0` by default.

## `level`
**Applicable items:** `enchanted_book`.

Returns the level of the book's enchantment.
Behaviour is undefined on books with multiple enchantments.

> [!TIP]
>
> The `stored_enchantment` module has built-in support for enchantment levels.
