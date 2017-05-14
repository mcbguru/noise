
## Limit Clause

To limit the number of results, use a limit clause at the end of the query.

This limits the results to the first 10 found:

```Thrift
find {foo: == "bar"}
return .baz
limit 10
```
