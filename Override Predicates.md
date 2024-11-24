# Override Predicates

Override predicates are the vanilla way of defining variants of a model, based on numerical data. Those can be used inside the identifier-based models managed by Variant-CIT's, in order to provide a second layer of variance to the item.

This mod currently only provides a single predicate.

## `bucket_entity_age`
**Applicable items:** all.

Returns the value of `Age` in the `bucket_entity_data` component, or `0` by default.

## `level`
**Applicable items:** `enchanted_book`.

Returns the level of the book's enchantment.
Behaviour is undefined on books with multiple enchantments.
