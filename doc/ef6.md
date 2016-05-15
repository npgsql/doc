---
layout: doc
title: Entity Framework 6
---

Npgsql has an Entity Framework 6 provider. You can use it by installing the
[EntityFramework6.Npgsql](https://www.nuget.org/packages/EntityFramework6.Npgsql/) nuget.

## Guid Support ##

Npgsql EF migrations support uses `uuid_generate_v4()` function to generate guids.
In order to have access to this function, you have to install the extension uuid-ossp through the following command:

{% highlight sql %}
create extension "uuid-ossp";
{% endhighlight %}

If you don't have this extension installed, when you run Npgsql migrations you will get the following error message:

{% highlight C# %}
ERROR:  function uuid_generate_v4() does not exist
{% endhighlight %}

If the database is being created by Npgsql Migrations, you will need to
[run the `create extension` command in the `template1` database](http://stackoverflow.com/a/11584751).
This way, when the new database is created, the extension will be installed already.
