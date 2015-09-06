---
layout: doc
title: Entity Framework 7
---

An experimental Npgsql Entity Framework 7 provider is available for testing.
Note that like EF7 itself the provider is under heavy development, but most of the basic features work.

To use the EF7 provider, install the latest prerelease version of NuGet
[`EntityFramework7.Npgsql`](https://www.nuget.org/packages/EntityFramework7.Npgsql/) from nuget.org.
You can then follow the "getting started"

General progress is tracked in the [EF7 issue on the npgsql github](https://github.com/npgsql/npgsql/issues/249).

Please let us know of any bugs you run across!

Large-scale features not yet implemented:

* CoreCLR ([#471](EF://github.com/npgsql/npgsql/issues/471))
* Reverse engineering (RevEng) - generating code model from an existing database
  ([#758](https://github.com/npgsql/npgsql/issues/758))


