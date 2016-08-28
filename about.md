---
layout: page
title: About
---
## General

Npgsql is an open source ADO.NET Data Provider for PostgreSQL, it allows programs written in C#, Visual Basic, F# to access the PostgreSQL database server.
It allows any program developed for .NET framework to access database server. It is implemented in 100% C# code. Works with PostgreSQL 9.x and above.

## PostgreSQL Compatibility

We aim to be compatible with all [currently supported PostgreSQL versions](http://www.postgresql.org/support/versioning/), which means 5 years back.
Earlier versions may still work but we don't perform continuous testing on them or commit to resolving issues on them.

For more compatibility information please see [this page](doc/compatibility.html).

## Non-Windows Platforms

Npgsql runs on .NET Core (netstandard13), and also on mono, and we run tests on all platforms in our CI process to keep it that way.
Please report any issues you find.

## Thanks

A special thanks to Rowan Miller, Scott Hanselman and Martin Woodward at Microsoft for generously donating an Azure subscription
for Npgsql's continuous integration platform.

## License

Npgsql is licensed under the [PostgreSQL License](https://github.com/npgsql/npgsql/blob/dev/LICENSE.txt), a liberal OSI-approved open source license.
