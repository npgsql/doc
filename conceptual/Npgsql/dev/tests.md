---
layout: page
title: Tests
---

## Overview

Npgsql has an extensive test suite to guard against regressions. The test suite is run on the official build server for the .NET Framework and .NET Core with all supported PostgreSQL backends.

Continuous integration results are publicly available via [Github Actions](https://github.com/npgsql/npgsql/actions).

## Getting Started

### Setup PostgreSQL

The Npgsql test suite requires a PostgreSQL backend for tests to run. By default, the test suite expects PostgreSQL to be running on the local machine with the default port (5432).

1. Install PostgreSQL: <https://www.postgresql.org/download>
2. Start the PostgreSQL backend.

### Create the `npgsql_tests` account

By default, the test suite expects an account named `npgsql_tests` with a password of `npgsql_tests`. This account is used by the test suite to create a database named `npgsql_tests` and run the tests.

```bash
$ psql -h localhost -U postgres
postgres=# CREATE USER npgsql_tests PASSWORD 'npgsql_tests' SUPERUSER;
postgres=# CREATE DATABASE npgsql_tests OWNER npgsql_tests;
```

_Note: superuser access is required to create and drop test databases, load extensions (e.g. `hstore`, `postgis`), etc._

### Clone the repository

```bash
cd ~
git clone git@github.com:npgsql/npgsql.git (use ssh)
git clone https://github.com/npgsql/npgsql.git (use https)
```

### Run the test suite

```bash
cd ~/npgsql
dotnet test ./test/Npgsql.Tests
dotnet test ./test/Npgsql.PluginTests
dotnet test ./test/Npgsql.Benchmarks
```
