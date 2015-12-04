---
layout: doc
title: Entity Framework 7
---

An experimental Npgsql Entity Framework 7 provider is available for testing.
Note that like EF7 itself the provider is under heavy development, but most of the basic features work.

To use the EF7 provider, install the latest prerelease version of NuGet
[`EntityFramework7.Npgsql`](https://www.nuget.org/packages/EntityFramework7.Npgsql/) from nuget.org.
You can then follow [EF7's "getting started" section](http://ef.readthedocs.org/en/latest/getting-started/full-dotnet/index.html),
using Npgsql instead of SqlServer.

Reverse-engineering (database-first) is also supported; the Npgsql provider for that is
[`EntityFramework7.Npgsql.Design`](https://www.nuget.org/packages/EntityFramework7.Npgsql.Design/).
The database-first instructions in the EF7 getting started work, just change the provider name and the connection string.

General progress is tracked in the [EF7 issue on the npgsql github](https://github.com/npgsql/npgsql/issues/249).

Please let us know of any bugs you run across!

Large-scale features not yet implemented:

* Computed properties which get generated on update ([#759](https://github.com/npgsql/npgsql/issues/759))

