## Return Clause

The return clause is how data or scoring is returned to the client. You can extract the whole document, a single field, multiple fields, and perform aggregations.

For this section these examples the following document will be used:

```json
{
  "_id": "example",
  "foo": "bar",
  "baz": {"biz": "bar"},
  "faz": [
    {"fiz": 213},
    {"biz": 5463},
    {"biz": 73}
  ]
}
```

### Basic Dot Notation

A leading dot indicates the root of the document. To return the whole document, place a single dot in return clause.

This will return the whole document for each document found.

```Thrift
find
    {foo: == "bar"}
return
    .
// [{
//   "_id": "example",
//   "foo": "bar",
//   "baz": {"biz": "bar"},
//   "faz": [
//     {"fiz": 213},
//     {"biz": 5463},
//     {"biz": 73}
//   ]
// }]
```

To return a specific field, place the field name after the dot:

```Thrift
find {foo: == "bar"}
return .baz
// [{"biz": "bar"}]
```

To return a nested field, use another dot:

```Thrift
find {foo: == "bar"}
return .baz.biz
// ["bar"]
```

To return an array element, use the array notation:

```Thrift
find {foo: == "bar"}
return .faz[1]
// [{"biz": 5463}]
```

To return an object field nested in the array, add a dot after the array notation:

```Thrift
find {foo: == "bar"}
return .faz[1].biz
// [5463]
```

To return multiple values, embed the return paths in other JSON structures.

For each match this example returns 2 values inside an array:

```Thrift
find {foo: == "bar"}
return [.baz, .faz]
// [[
//   {"biz": "bar"},
//   [{"fiz": 213}, {"biz": 5463}, {"biz": 73}]
// ]]
```

For each match this example return 2 values inside an object:

```Thrift
find {foo: == "bar"}
return {baz: .baz, faz: .faz}
// [{
//   "baz": {"biz": "bar"},
//   "faz": [{"fiz": 213}, {"biz": 5463}, {"biz": 73}]
// }]
```

### Missing Values

Sometimes you'll want to return a field that doesn't exist on a matching document. When that happens, `null` is returned.

If you'd like a different value to be returned, use the `default=<json>` option, like this:

```Thrift
find {foo: == "bar"}
return .hammer default=0
// [0]
```

Each returned value can have a default as well.

```Thrift
find {foo: == "bar"}
return {baz: .baz default=0, hammer: .hammer default=1}
// [{
//   "baz": {"biz": "bar"},
//   "hammer": 1
// }]
```



### Return a Field from All Objects Inside an Array

If want to return a nested field inside an array, but for each object in the array, use the `[]` with no index.

This will return each biz field as an array of values:

```Thrift
find {foo: == "bar"}
return .faz[].biz
// [[5463, 73]]
```

### Bind Variables: Return Only Matched Array Elements

If you are searching for nested values or objects nested in arrays, and you want to return only the match objects, use the bind syntax before the array in the query. The bound value is always an array, as multiple elements might match.

Say you have a document like this:

```json
{
  "_id": "a",
  "foo": [
    {"fiz": "bar", "val": 4}, {"fiz": "baz", "val": 7}
  ],
  "bar": [
    {"fiz": "baz", "val": 9}
  ]
}

```

You want to return the object where `{"fiz": "bar", ...}` (but not the others), use you a bind variable (`var::[...]`), like this:

```Thrift
find {foo: x::[{fiz: == "bar"}]}
return x
// [[{"fiz": "bar", "val": 4}]]
```

If instead you want to return the `val` field, add the `.val` to the bind variable like this:

```Thrift
find {foo: x::[{fiz: == "bar"}]}
return x.val
// [[4]]
```

You can have any number of bind variables:

```Thrift
find {foo: x::[{fiz: == "bar"}], foo: y::[{fiz: == "baz"}]}
return [x.val, y.val]
// [[[4], [7]]]
```

The same query as the previous one, but returning an object:

```Thrift
find {foo: x::[{fiz: == "bar"}], foo: y::[{fiz: == "baz"}]}
return {x: x.val, y: y.val}
// [{"x": [4], "y": [7]}]
```

You can reuse bind variables in different clauses and they'll be combined:

```Thrift
find {foo: x::[{fiz: == "baz"}] || bar: x::[{fiz: == "baz"}]}
return {x: x.val}
// [{"x": [7, 9]}]
```
