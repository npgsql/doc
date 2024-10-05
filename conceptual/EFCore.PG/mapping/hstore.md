# `hstore` Type Mapping

PostgreSQL has the unique feature of supporting [*hstore data types*](https://www.postgresql.org/docs/current/static/hstore.html). This allows you to conveniently and efficiently store a set of string key/value pairs in a single column, where in other database you'd typically resort to storing the values as a JSON string or defining another table with a one-to-many relationship.

## Mapping hstores

Define a `Dictionary<string, string>` or `ImmutableDictionary<string, string>` property on an entity and it will automatically use the `hstore` value type.
The next migration you add will also enable the `hstore` extension in PostgreSQL for you automatically.

The `hstore` type does also support nullable columns so you can define a `Dictionary<string, string>?` or `ImmutableDictionary<string, string>?` to make the column nullable.

```csharp
public class Post
{
    public int Id { get; set; }
    public Dictionary<string, string> Metadata { get; set; } = new();
    public ImmutableDictionary<string, string> Tags { get; set; } = ImmutableDictionary<string, string>.Empty;
    public Dictionary<string, string>? Events { get; set; }
    public ImmutableDictionary<string, string>? Labels { get; set; }
}
```

The provider will create `hstore` columns for the above four properties, and will properly detect changes in them.

If you load a `Dictionary<string, string>` and change one of its elements, calling `SaveChanges` will automatically update the row in the database accordingly.
As an `ImmutableDictionary<string, string>` is immutable, you must `set` the property to a new value when adding/updating values and then call `SaveChanges` to update the database row.

```csharp
post.Tags = post.Tags.Add("a key", "a value");
```

## Operation translation

The provider can also translate CLR Dictionary operations to the corresponding SQL operation; this allows you to efficiently work with `hstore`s by evaluating operations in the database and avoids pulling all the data. The following table lists the Dictionary operations that currently get translated; all these translations work both for .NET `Dictionary<string, string>` and `ImmutableDictionary<string, string>` unless specifically noted. If you run into a missing operation, please open an issue.

 .NET                          | SQL                                                                                                                                     | Notes                                                                                                        
-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------
 store[key]                    | [store -> key](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                     |
 store.Count                   | [cardinality(akeys(store))](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                        |
 store.Count()                 | [cardinality(akeys(store))](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                        | Works on `IEnumerable<KeyValuePair<string, string>>` to count concatenated or subtracted stores. 
 store1 == store2              | [store1 = store2](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                  |
 store1.SequenceEqual(store2)  | [store1 = store2](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                  | Compatible with columns mapped as `jsonb` 
 store.ContainsKey(key)        | [store ? key](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                      | Compatible with columns mapped as `jsonb` 
 store.Remove(key)             | [store - key](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                      | Only `ImmutableDictionary<string, string>`. Compatible with columns mapped as `jsonb` 
 store.ContainsValue(value)    | [value = ANY(avals(store))](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                        |
 store.IsEmpty                 | [cardinality(akeys(store)) = 0](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                    | Only `ImmutableDictionary<string, string>`                                                                   
 store.Any()                   | [cardinality(akeys(store)) <> 0](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                   |
 store.ToDictionary()          | [store](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                    | Converts an `IEnumerable<KeyValuePair<string, string>>` to a `Dictionary<string, string>`. Compatible with columns mapped as `json` and `jsonb`                     
 store.ToImmutableDictionary() | [store](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                    | Converts an `IEnumerable<KeyValuePair<string, string>>` to an `ImmutableDictionary<string, string>`. Compatible with columns mapped as `json` and `jsonb` 
 store1.Concat(store2)         | [store1 \|\| store2](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                               | Call `ToDictionary()` or `ToImmutableDictionary()` on result. Compatible with columns mapped as `json` and `jsonb`                                                 
 store1.Except(store2)         | [store1 - store2](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                  | Call `ToDictionary()` or `ToImmutableDictionary()` on result. Compatible with columns mapped as `json` and `jsonb`                                                 
 store.Keys                    | [akeys(store)](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                     | Only `ImmutableDictionary<string, string>`                                                                   
 store.Keys.ToList()           | [akeys(store)](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                     |
 store.Values                  | [avals(store)](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                     | Only `ImmutableDictionary<string, string>`                                                                   
 store.Values.ToList()         | [avals(store)](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                     |
 store1.Keys.Concat(store2.Keys.Concat) | [akeys(store1) \|\| akeys(store2) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                |
 store1.Values.Concat(store2.Values.Concat) | [avals(store1) \|\| avals(store2) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                |
 EF.Functions.ValuesForKeys(store, keys) | [store -> keys](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                    | Compatible with columns mapped as `jsonb` 
 EF.Functions.ContainsAllKeys(store, keys) | [store ?& keys](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                    | Compatible with columns mapped as `jsonb` 
 EF.Functions.ContainsAnyKeys(store, keys) | [store ?\| keys](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                   | Compatible with columns mapped as and `jsonb` 
 EF.Functions.Contains(store1, store2) | [store1 @> store2 ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                |  Accepts two `IEnumerable<KeyValuePair<string, string>>`. Compatible with columns mapped as `jsonb` 
 EF.Functions.ContainedBy(store1, store2) | [store1 <@ store2 ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                |  Accepts two `IEnumerable<KeyValuePair<string, string>>`. Compatible with columns mapped as `jsonb` 
 EF.Functions.Remove(store, key) | [store - key](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                      | Compatible with columns mapped as `jsonb` 
 EF.Functions.Slice(store, keys) | [ slice(store, keys) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                           | Compatible with columns mapped as `json` and `jsonb` 
 EF.Functions.ToKeysAndValues(store) | [hstore_to_array](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                                | Compatible with columns mapped as `json` and `jsonb` 
 EF.Functions.FromKeysAndValues(keysAndValues) | [ hstore(keysAndValues) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                        |
 EF.Functions.FromKeysAndValues(keys, values) | [ hstore(keys, values) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                         |
 EF.Functions.ToJson(store) | [ hstore_to_json(store) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                        |
 EF.Functions.ToJsonb(store) | [ hstore_to_jsonb(store) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                       |
 EF.Functions.ToJsonLoose(store) | [ hstore_to_json_loose(store) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                  |
 EF.Functions.ToJsonbLoose(store) | [ hstore_to_jsonb_loose(store) ](https://www.postgresql.org/docs/current/hstore.html#HSTORE-OPS-FUNCS)                                 |
 EF.Functions.FromJson(json) | [ select hstore(array_agg(key), array_agg(value)) FROM json_each_text(json) ](https://www.postgresql.org/docs/9.3/functions-json.html)  | Not natively supported in PostgreSQL
 EF.Functions.FromJsonb(json) | [ select hstore(array_agg(key), array_agg(value)) FROM jsonb_each_text(json) ](https://www.postgresql.org/docs/9.3/functions-json.html) | Not natively supported in PostgreSQL


