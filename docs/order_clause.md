## Order Clause

To return results in a particular order, use the order clause.

This will order results ascending based on the contents of the `baz` field:

```
find {foo: == "bar"}
order .baz
```

If `baz` doesn't existing, `null` be the value used for ordering.

This will order `baz` descending:

```
find {foo: == "bar"}
order .baz
```

This will order `baz` ascending:

```
find {foo: == "bar"}
order .baz asc
```

This will order `baz` ascending with default value of `1` if no `baz` value exists:

```
find {foo: == "bar"}
order .baz asc default=1
```

This will order `baz` ascending, for values of `baz` that are the same, those results are now ordered as `biz` ascending.

```
find {foo: == "bar"}
order .baz asc, .biz dsc
```

