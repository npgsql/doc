---
layout: post
author: roji
title:  "Porting to dotnet RC2 and Restructuring"
date:   2016-04-13
---
After three months of inactivity due to a super-intensive project, I'm back to life.
Here is some info on some work done on the project.

Npgsql has been ported to the latest dotnet RC2 bits and compiles with the dotnet CLI
(no more DNX). 

The Entity Framework Core provider (previously EF7) is being moved into
[its own repository](https://github.com/npgsql/Npgsql.EntityFrameworkCore.PostgreSQL).
There are several reasons for this change:

* The general instability of EFCore caused frequent build issues which interfered with
  unrelated work on Npgsql.
* The EFCore provider and Npgsql are very loosely-coupled, there's no real reason for
  them to coexist in the same repo.
* The EFCore provider needs to follow its own release and versioning rhythm. Since we
  use [gitversion](https://github.com/GitTools/GitVersion), all projects within the
  same git repo follow the same versioning; note how the EF6 provider was "released"
  every time Npgsql was released, even if no EF6 change happened.
* There's no reason to run all Npgsql tests if an EF7-only change occurs, this
  needlessly slows down our build in continuous integration.
* Ability to give people permissions to the EFCore project only, etc.

Note that the EF Core provider hasn't been ported to RC2 yet and is in a broken state,
I will get around to this soon.
Note also that a similar decoupling of the EF6 provider into its own repo will probably
happen soon as well.

Finally, Npgsql also uses a complex process to rewrite synchronous methods into async counterparts,
using a package called AsyncRewriter. This process frequently broke down and caused issues
for people trying to build the project, and slowed down the build as well. As of now the
resulting GeneratedAsync.cs is committed into git, so building should be much easier. You
still can (and occasionally must) trigger AsyncRewriter by dropping down to shell and
executing `dotnet rewrite-async`.

The following weeks will be dedicated to working on Npgsql itself and getting to a 3.1 beta ASAP.

Please let me know about any issues you run into!
