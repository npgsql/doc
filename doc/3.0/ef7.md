---
layout: doc-3.0
title: Entity Framework 7
---

An experimental Npgsql Entity Framework 7 provider is available for testing.
Note that like EF7 itself the provider is under heavy development, but most of the basic features work.

Development happens in the
[Npgsql.EntityFrameworkCore.PostgreSQL](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL) repo,
all issues should be opened there. RC2 work is in progress but the provider isn't ready yet, use RC1 as instructed
below.

Reverse-engineering (database-first) is also supported; the Npgsql provider for that is
[`EntityFramework7.Npgsql.Design`](https://www.nuget.org/packages/EntityFramework7.Npgsql.Design/).
The database-first instructions in the EF7 getting started work, just change the provider name and the connection string.

General progress is tracked in the [EF7 issue on the npgsql github](https://github.com/npgsql/npgsql/issues/249).

Please let us know of any bugs you run across!

Large-scale features not yet implemented:

* Computed properties which get generated on update ([#759](https://github.com/npgsql/npgsql/issues/759))

