## Find Clause

All queries have a `find` clause followed by an example based query syntax. It is a combination of expressions that consist of three parts: The key to query, an operator and the value to match.

This query will return the `_id` of every document with a `{"foo": "bar",...}`

```
find {foo: == "bar"}
```

This query will match all documents in the index and return their `_id`.

```
find {}
```

To match on multiple fields, or even nested fields, simply construct the same json structure in query form.

Match on two fields:

```
find {foo: == "bar", fizz: == "buzz"}
```

Match on fields, one nested within another:

```
find {foo: == "bar", fizz: {fazz: == "buzz"}}
```

### Word Match Operator

`~=` is the full text match operator. Use it find a word in a text field.

```
find {body: ~= "word"}
```

Put multiple words in the quoted string to find a phrase in the field.

```
find {body: ~= "a multi word sentence"}
```

To find words that are within a specified distance of each other, put the the maximum word distance in the operator. This example will return results where each word is with the 50 words of the others.

```
find {body: ~50= "bitcoin gold price"}
```

### Comparison Operators

Noise supports the following comparison operators:

|Operator|Description|Types
---------|-----------|-----
|`==`|Equality|Strings, Numbers, true, false, null
|`>`|Less Than|Numbers
|`<`|Greater Than|Numbers
|`>=`|Less Than or Equal|Numbers
|`<=`|Greater Than or Equal|Numbers

Noise does not do type conversions of datatypes. Strings only compare with strings, number only compare with numbers, etc.

### Finding Things in Arrays

Let's say you have document like this with text in an array:

```
{"foo": ["bar", "baz"]}
```

To find element with value `"baz"` in the array, use syntax like this:

```
find {foo:[ == "baz"]}
```

If objects are nested in array, like this:

```
{"foo": [{"fiz": "bar"}, {"fiz": "baz"}]}
```

To find a `{"fiz": "baz"}` in the array, use syntax like this:

```
find {foo: [{fiz: == "baz"}]}
```

### Boolean Logic and Parens

Noise has full support for boolean logic using `&&` (logical AND) and `||` (logical OR) operators and nesting logic with parens.

The comma `,` in objects is actually the same as the `&&` operator. They can be used interchangeably for which ever is more readable.

Find a doc with `"foo"` or `"bar"` in the `body`:

```
find {body: ~= "foo" || body: ~= "bar"}
```

Find a doc that has `"foo"` or `"bar"` and has `"baz"` or `"biz"` in the `body`:

```
find {(body: ~= "foo" || body: ~= "bar") &&
      (body: ~= "baz" || body: ~= "biz")}
```

The fields can be nested as well. Find a doc where either the nested field `fiz` contains either `"baz"` or `"biz"`.

```
find {foo: {fiz: ~= "baz" || fiz: ~= "biz"}}
```


### Not Operator

Use the `!` (logical NOT) to exclude matching criteria.

Find docs where `foo` has value `"bar"` and `fab` does not have value `"baz"`:

```
find {foo: == "bar", fab: !== "baz"}
```

You can use logical not with parentheses to negate everything enclosed. This example finds docs where `foo` has value `"bar"` and `fab` does not have value `"baz"` or `"biz"`':

```
find {foo: == "bar", !(fab: == "baz" || fab: == "biz")}
```

You cannot have every clause be negated. Query need at least one non-negated clauses.

Illegal:

```
find {foo: !~= "bar" && foo: !~= "baz"}
```

Illegal:

```
find {!(foo: ~= "bar" && foo: ~= "baz"})
```

Also double negation is not allowed.

Illegal:

```
find {foo ~= "waz" && !(foo: ~= "bar" && foo: !~= "baz"})
```

### Relevancy Scoring and Boosting

Relevancy scoring uses a combination boolean model and Term Frequency/Inverse Document Frequency (TF/IDF) scoring system, very similar to Lucene and Elastic Search. The details of the scoring model is beyond the scope of the document.

To return results in relevancy score order (most relevant first), simply use the order clause with the `score()` function.

```
find {subject: ~= "hammer" || body: ~= "hammer"}
order score() desc
```

But if you want matches in `subject` fields to score higher than in `body` fields, you can boost the score with the `^` operator. It is a multiplier of the scores of associated clauses.

This boosts `subject` matches by 2x:

```
find {subject: ~= "hammer"^2 || body: ~= "hammer"}
order score() desc
```

You can also boost everything in parenthesis or objects or arrays:

```
find {(subject: ~= "hammer" || subject: ~= "nails")^2 ||
       body: ~= "hammer" ||  body: ~= "nails"}
order score() desc
```
Another way to express the same thing:

```
find {subject: ~= "hammer" || subject: ~= "nails"}^2 ||
     {body: ~= "hammer" || body: ~= "nails"}
order score() desc
```
