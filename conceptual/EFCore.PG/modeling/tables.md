# Tables

## Naming

By default, EF Core will map to tables and columns named exactly after your .NET classes and properties, so an entity type named `BlogPost` will be mapped to a PostgreSQL table called `BlogPost`. While there's nothing wrong with that, the PostgreSQL world tends towards snake_case naming instead. In addition, any upper-case letters in unquoted identifiers are automatically converted to lower-case identifiers, so the Npgsql provider generates quotes around all such identifiers.

Starting with 3.0.0, you can use the [EFCore.NamingConventions](https://github.com/efcore/EFCore.NamingConventions) plugin to automatically set all your table and column names to snake_case instead:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseNpgsql(...)
        .UseSnakeCaseNamingConvention();

public class Customer {
    public int Id { get; set; }
    public string FullName { get; set; }
}
```

This will cause cleaner SQL such as the following to be generated:

```sql
CREATE TABLE customers (
            id integer NOT NULL GENERATED BY DEFAULT AS IDENTITY,
            full_name text NULL,
            CONSTRAINT "PK_customers" PRIMARY KEY (id);

SELECT c.id, c.full_name
        FROM customers AS c
        WHERE c.full_name = 'John Doe';
```

See the [plugin documentation](https://github.com/efcore/EFCore.NamingConventions) for more details,

## Storage parameters

PostgreSQL allows configuring tables with *storage parameters*, which can tweak storage behavior in various ways; [see the PostgreSQL documentation](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS) for more information.

To configure a storage parameter on a table, use the following code:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
    => modelBuilder.Entity<Blog>().HasStorageParameter("fillfactor", 70);
```
