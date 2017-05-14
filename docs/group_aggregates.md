## Grouping and Aggregation

Noise includes ways to group rows together and aggregate values.

Values you want to group together use `group(...)` function in the `return` clause.

For values that are grouped together you can then perform aggregations on other values and return that aggregation. If a group function is used, all other fields must also be grouped or aggregated.

The aggregation functions available are:

|function      | Description|
---------------|-------------
|`array(...)`|Returns all values in the group as values in an array.|
|`array_flat(...)`|Returns all values in the group as values in an array. However if an array is encountered it extracts all the values inside the array (and further nested arrays) and returns them as a singe flat array|
|`avg(...)`|Averages numeric values in the group. If numeric values are in arrays, it extracts the values from the arrays. Even if arrays are nested in arrays, it extracts through all levels of nested arrays and averages them. |
|`count()`| Returns the count of the grouped rows for each grouping. |
|`concat(... [sep="..."])`| Returns all the strings in the group as a single concatenated string. Other value types are ignored. Use the optional `sep="..."` to specify a separator between string values.|
|`max(...)`|Returns the maximum value in the group. See type ordering below to see how different types are considered. |
|`max_array(...)`|Returns the maximum value in the group, if array is encountered the values inside the array are extracted and considered.|
|`min(...)`|Returns the minimum value in the group. See type ordering below to see how different types are considered.|
|`min_array(...)`|Returns the minimum value in the group, if array is encountered the values inside the array are extracted and considered.|
|`sum(...)`|Sums numeric values in the group. If numeric values are in arrays, it extracts the values from the arrays. Even if arrays are nested in arrays, it extracts through all levels of nested arrays and sums them.|

To perform grouping and/or aggregate, each field returned will need either a grouping or a aggregate function. It's an error it on some returned fields but not others.

Groupings are are ordered first on the leftmost `group(...)` function, then on the next leftmost, etc.

You do not need to use `group(...)` to perform aggregates. If you have no `group(...)` defined, then all rows are aggregated into a single row.



### Max/Min Type Ordering
The ordering of types for `max(...)` and `min(...)` is as follows:

null < false < true < number < string < array < object


## Group/Aggregate Examples


Let's say we have documents like this:

```json
{"foo":"group1", "baz": "a", "bar": 1}
{"foo":"group1", "baz": "b", "bar": 2}
{"foo":"group1", "baz": "c", "bar": 3}
{"foo":"group1", "baz": "a", "bar": 1}
{"foo":"group1", "baz": "b", "bar": 2}
{"foo":"group1", "baz": "c", "bar": 3}
{"foo":"group1", "baz": "a", "bar": 1}
{"foo":"group1", "baz": "b", "bar": 2}
{"foo":"group1", "baz": "c", "bar": 3}
{"foo":"group1", "baz": "a", "bar": 1}
{"foo":"group1", "baz": "b", "bar": 2}
{"foo":"group1", "baz": "c", "bar": 3}
{"foo":"group2", "baz": "a", "bar": "a"}
{"foo":"group2", "baz": "a", "bar": "b"}
{"foo":"group2", "baz": "b", "bar": "a"}
{"foo":"group2", "baz": "b", "bar": "b"}
{"foo":"group2", "baz": "a", "bar": "a"}
{"foo":"group2", "baz": "a", "bar": "c"}
{"foo":"group2", "baz": "b", "bar": "d"}
{"foo":"group2", "baz": "b", "bar": "e"}
{"foo":"group2", "baz": "a", "bar": "f"}
{"foo":"group3", "baz": "a", "bar": "a"}
("foo":"group3",             "bar": "b"}
{"foo":"group3", "baz": "b", "bar": "a"}
{"foo":"group3", "baz": "b", "bar": "b"}
{"foo":"group3", "baz": "a", "bar": "a"}
{"foo":"group3", "baz": "a"            }
{"foo":"group3", "baz": "b", "bar": "d"}
{"foo":"group3", "baz": "b", "bar": "e"}
{"foo":"group3", "baz": "a", "bar": "f"}
```

### Count

Query:
```
find {foo: == "group1"}
return {baz: group(.baz), count: count()}
```
Results:

```json
{"baz":"a","bar":4}
{"baz":"b","bar":4}
{"baz":"c","bar":4}

```

### Sum

Query:

```
find {foo: == "group1"}
return {baz: group(.baz), bar: sum(.bar)}
```

Results:

```json
{"baz":"a","bar":4}
{"baz":"b","bar":8}
{"baz":"c","bar":12}

```

### Avg

Query:

```
find {foo: == "group1"}
return {avg: avg(.bar)}
```

Results:

```json
{"bar":2}
```

### Concat

Query:

```
find {foo: == "group1"}
return {baz: group(.baz), concat: concat(.baz sep="|")}
```

Results:

```json
{"baz":"a","concat":"a|a|a|a"}
{"baz":"b","concat":"b|b|b|b"}
{"baz":"c","concat":"c|c|c|c"}
```

### Max

Query:

```
find {foo: == "group1"}
return {max: max(.bar)}
```
Results:

```json
{"max":3}
```

Query:

```
find {foo: == "group1"}
return {max: max(.baz)}
```

Results:

```json
{"max":"c"}
```

### Min

Query:

```
find {foo: == "group1"}
return {min: min(.bar)}
```

Results:

```json
{"min":1}
```

### Group Ordering

Query:

```
find {foo: == "group2"}
return [group(.baz order=asc), group(.bar order=desc), count()]
```

Results:

```json
["a","f",1]
["a","c",1]
["a","b",1]
["a","a",2]
["b","e",1]
["b","d",1]
["b","b",1]
["b","a",1]
```

### Default Values

Query:

```
find {foo: =="group2"}
return [group(.baz order=asc) default="a", group(.bar order=desc) default="c", count()];
```

Results:

```json
["a","f",1]
["a","c",1]
["a","b",1]
["a","a",2]
["b","e",1]
["b","d",1]
["b","b",1]
["b","a",1]
```

### Arrays

When performing aggregations on arrays, some functions will extract values out of the arrays (and arrays nested in arrays).

We have documents like this:

```json
{"foo":"array1", "baz": ["a","b",["c","d",["e"]]]}
{"foo":"array1", "baz": ["f","g",["h","i"],"j"]}
{"foo":"array2", "baz": [1,2,[3,4,[5]]]}
{"foo":"array2", "baz": [6,7,[8,9],10]};
```

Query:

```
find {foo: == "array1"}
return array(.baz)
```

Results:

```json
[["f","g",["h","i"],"j"],["a","b",["c","d",["e"]]]]
```

Query:

```
find {foo: == "array1"}
return array_flat(.baz)
```

Results:

```json
["f","g","h","i","j","a","b","c","d","e"]
```

Query:

```
find {foo: == "array1"}
return max(.baz)
```

Results:

```json
["f","g",["h","i"],"j"]
```

Query:

```
find {foo: == "array1"}
return max_array(.baz)
```

Results:

```json
"j"
```

Query:

```
find {foo: == "array1"}
return min_array(.baz)
```

Results:

```json
"a"
```

Query:

```
find {foo: =="array2"}
return avg(.baz)
```

Results:

```json
5.5
```

Query:

```

find {foo: =="array2"}
return sum(.baz)
```

Results:

```json
55
```
