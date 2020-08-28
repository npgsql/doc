---
layout: doc
title: Entity Framework 6
---

Npgsql has an Entity Framework 6 provider. You can use it by installing the
[EntityFramework6.Npgsql](https://www.nuget.org/packages/EntityFramework6.Npgsql/) nuget.

## Basic Configuration ##

Configuration for an Entity Framework application can be specified in a config file (app.config/web.config) or through code. The latter is known as code-based configuration.

### Code-based ###

To use Entity Framework with Npgsql, define a class that inherits from `DbConfiguration` in the same assembly as your class inheriting `DbContext`. Ensure that you configure provider services, a provider factory, a default connection factory as shown below:

```csharp
using Npgsql;
using System.Data.Entity;

class NpgSqlConfiguration : DbConfiguration
{
    public NpgSqlConfiguration()
    {
        var name = "Npgsql";

        SetProviderFactory(providerInvariantName: name,
        providerFactory: NpgsqlFactory.Instance);

        SetProviderServices(providerInvariantName: name,
        provider: NpgsqlServices.Instance);

        SetDefaultConnectionFactory(connectionFactory: new NpgsqlConnectionFactory());
    }
}
```

### Config file ###

When installing `EntityFramework6.Npgsql` nuget package, the relevant sections in `App.config` / `Web.config` are usually automatically updated. You typically only have to add your `connectionString` with the correct `providerName`.

```xml
<configuration>
    <connectionStrings>
        <add name="BlogDbContext" connectionString="Server=localhost;port=5432;Database=Blog;User Id=postgres;Password=postgres;" providerName="Npgsql" />
    </connectionStrings>
    <entityFramework>
        <providers>
            <provider invariantName="Npgsql" type="Npgsql.NpgsqlServices, EntityFramework6.Npgsql" />
        </providers>
        <!-- setting the default connection factory is optional -->
        <defaultConnectionFactory type="Npgsql.NpgsqlConnectionFactory, EntityFramework6.Npgsql" />
    </entityFramework>
    <system.data>
        <DbProviderFactories>
            <add name="Npgsql Provider" invariant="Npgsql" description=".NET Framework Data Provider for PostgreSQL" type="Npgsql.NpgsqlFactory, Npgsql, Version=4.1.3.0, Culture=neutral, PublicKeyToken=5d8b90d52f46fda7" />
        </DbProviderFactories>
    </system.data>
</configuration>
```

## Guid Support ##

Npgsql EF migrations support uses `uuid_generate_v4()` function to generate guids.
In order to have access to this function, you have to install the extension uuid-ossp through the following command:

```sql
create extension "uuid-ossp";
```

If you don't have this extension installed, when you run Npgsql migrations you will get the following error message:

```
ERROR:  function uuid_generate_v4() does not exist
```

If the database is being created by Npgsql Migrations, you will need to
[run the `create extension` command in the `template1` database](http://stackoverflow.com/a/11584751).
This way, when the new database is created, the extension will be installed already.

## Template Database ##

When the Entity Framework 6 provider creates a database, it issues a simple `CREATE DATABASE` command.
In PostgreSQL, this implicitly uses `template1` as the template - anything existing in `template1` will
be copied to your new database. If you wish to change the database used as a template, you can specify
the `EF Template Database` connection string parameter. For more info see the
[PostgreSQL docs](https://www.postgresql.org/docs/current/static/sql-createdatabase.html).

## Customizing DataReader Behavior ##

You can use [an Entity Framework 6 IDbCommandInterceptor](https://msdn.microsoft.com/en-us/library/dn469464(v=vs.113).aspx) to wrap the `DataReader` instance returned by Npgsql when Entity Framework executes queries. This is possible using a ```DbConfiguration``` class.

Example use cases:
- Forcing all returned ```DateTime``` and ```DateTimeOffset``` values to be in the UTC timezone.
- Preventing accidental insertion of DateTime values having ```DateTimeKind.Unspecified```.
- Forcing all postgres date/time types to be returned to Entity Framework as ```DateTimeOffset```.

```c#
[DbConfigurationType(typeof(AppDbContextConfiguration))]
public class AppDbContext : DbContext
{
    // ...
}

public class AppDbContextConfiguration : DbConfiguration
{
    public AppDbContextConfiguration()
    {
        this.AddInterceptor(new MyEntityFrameworkInterceptor());
    }
}

class MyEntityFrameworkInterceptor : DbCommandInterceptor
{
    public override void ReaderExecuted(
        DbCommand command,
        DbCommandInterceptionContext<DbDataReader> interceptionContext)
    {
        if (interceptionContext.Result == null) return;
        interceptionContext.Result = new WrappingDbDataReader(interceptionContext.Result);
    }

    public override void ScalarExecuted(
        DbCommand command,
        DbCommandInterceptionContext<object> interceptionContext)
    {
        interceptionContext.Result = ModifyReturnValues(interceptionContext.Result);
    }

    static object ModifyReturnValues(object result)
    {
        // Transform and then
        return result;
    }
}

class WrappingDbDataReader : DbDataReader, IDataReader
{
    // Wrap an existing DbDataReader, proxy all calls to the underlying instance, 
    // modify return values and/or parameters as needed...
    public WrappingDbDataReader(DbDataReader reader)
    {
    }
}
```
