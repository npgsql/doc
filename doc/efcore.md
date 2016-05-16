---
layout: doc
title: Entity Framework Core
redirect_from:
  - /doc/ef7.html
---

## Installation and General Info

An experimental Npgsql Entity Framework Core provider is available for testing.
Note that like EFCore itself the provider is under heavy development, but most of the basic features work.

The main provider package is
[Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/).
Reverse-engineering (database-first) is also supported; the provider for that is
[Npgsql.EntityFrameworkCore.PostgreSQL.Design](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.Design/).
The database-first instructions in the EFCore getting started work, just change the provider name and the connection string.

Development happens in [this github repo](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL), issues should be opened there.

Please let us know of any bugs you run across!

Features not yet implemented:

* Optimistic concurrency ([#19](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL/issues/19)).
* It's not possible to use PostgreSQL-specific types inside where clauses
  (https://github.com/aspnet/EntityFramework/issues/5365).
* Composite types aren't supported yet ([#22](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL/issues/22)).

---

## Cool Features

### Regular expressions

PostgreSQL supports
[regular expression operations in the database](http://www.postgresql.org/docs/current/static/functions-matching.html#FUNCTIONS-POSIX-REGEXP),
and the Npgsql EFCore provider provides some support for evaluating C# regex operations at the backend.
All you have to do is use Regex.IsMatch in your where clause:

{% highlight C# %}
var customersStartingWithA = context.Customers.Where(c => Regex.IsMatch(c.CompanyName, "^A"));
{% endhighlight %}

Since this regular expression is evaluated at the server, the EFCore provider doesn't need to load all
the customers from the database, saving lots of transfer bandwidth.

### Setting up PostgreSQL extensions

The provider allows you to specify PostgreSQL extensions that should be set up in your database.
Simply use HasPostgresExtension in your context's OnModelCreating:

{% highlight C# %}
protected override void OnModelCreating(ModelBuilder modelBuilder) {
    modelBuilder.HasPostgresExtension("hstore");
}
{% endhighlight %}

### Using a database template

When creating a new database,
[PostgreSQL allows specifying another "template database"](http://www.postgresql.org/docs/current/static/manage-ag-templatedbs.html)
which will be copied as the basis for the new one. You can trigger this by using HasDatabaseTemplate in your context's
OnModelCreating:

{% highlight C# %}
protected override void OnModelCreating(ModelBuilder modelBuilder) {
    modelBuilder.HasDatabaseTemplate("my_template_db");
}
{% endhighlight %}

