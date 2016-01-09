---
layout: page
title: Getting Npgsql
---
The best way to install Npgsql in your project is with our <a href="https://www.nuget.org/packages/Npgsql/">nuget package, Npgsql</a>.

- For Entity Framework 6, install <a href="https://www.nuget.org/packages/EntityFramework6.Npgsql/">EntityFramework6.Npgsql</a>
(In Npgsql 2.2 and earlier the package was <a href="https://www.nuget.org/packages/Npgsql.EntityFramework/">Npgsql.EntityFramework</a>.
- For Entity Framework 5, install <a href="https://www.nuget.org/packages/EntityFramework5.Npgsql/">EntityFramework5.Npgsql</a>
(or <a href="https://www.nuget.org/packages/Npgsql.EntityFrameworkLegacy/">Npgsql.EntityFrameworkLegacy</a> in Npgsql 2.2).
- An Entity Framework 7 provider is in the works but is still unsuitable for actual use. You can try it out via our unstable
feed (see below), package name EntityFramework7.Npgsql.

In some cases you'll want to install Npgsql into your
[Global Assembly Cache (GAC)](https://msdn.microsoft.com/en-us/library/yf1d93sz%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396).
This is usually the case when you're using a generic database program that can work with any ADO.NET provider but doesn't come
with Npgsql or reference it directly. For these cases, you can download the Npgsql Windows installer from
[our Github releases page](https://github.com/npgsql/npgsql/releases): it will install Npgsql (and optionally the Entity Framework
providers) into your GAC and add Npgsql's DbProviderFactory into your `machine.config` file.
This is *not* the general recommended method of using Npgsql - install via Nuget if possible.

If you'd like to have Visual Studio Design-Time support, you can try our <a href="https://github.com/npgsql/npgsql/releases">experiental installer</a> Setup_NpgsqlDdexProvider.exe.
And then follow the <a href="doc/ddex.html">instructions</a> in the documentation.

Our build server publishes CI nuget packages for every build. If a bug affecting you was fixed but there hasn't yet been a patch release,
you can get a CI nuget at our [stable MyGet feed](https://www.myget.org/gallery/npgsql). These packages are generally stable and
safe to use (although it's better to wait for a release).

We also publish CI packages for the next minor/major version at our [unstable MyGet feed](https://www.myget.org/gallery/npgsql-unstable).
These are definitely unstable and should be used with care.
