<a id="dev-entities-partial-indexes"></a>

# Partial Indexes

To use a partial index for the entity field, add the following condition as an additional option to the index definition:

```none
$table->addIndex(['is_featured'], 'idx_oro_product_featured', [], ['where' => '(is_featured = true)']);
```
