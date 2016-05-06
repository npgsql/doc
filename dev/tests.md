---
layout: page
title: Tests
---

## Overview

Npgsql comes with an extensive test suite to make sure no regressions occur. All tests are run on our build server on all supported .NET versions (including a recent version of mono) and all supported Postgresql backends.

There is also a growing suite of speed tests to be able to measure performance. These tests are currently marked [Explicit] and aren't executed automatically.

## Simple setup

The Npgsql test suite requires a PostgreSQL backend to test against. Simply use the latest version of Postgresql on your dev machine on the default port (5432).
By default, all tests will be run using user *npgsql_tests*, and password *npgsql_tests*. Npgsql will automatically create a database called *npgsql_tests* and
run its tests against this.

To set this up, connect to PostgreSQL as the admin user as follows:

{% highlight sql %}
psql -h localhost -U postgresql
<enter the admin password>
create user npgsql_tests password 'npgsql_tests' superuser;
{% endhighlight %}

And you're done.

Superuser access is needed for some tests, e.g. loading the hstore extension, creatig and dropping test databases in the Entity Framework tests...

## Testing against multiple backends

The test suite uses NUnit parameterized test fixtures to allow for testing against multiple backends. This allows our build server to make sure all tests complete against *all* supported backend major versions.

Setting this up is easy. The test suite checks for the existence of environment variables, which contain the connection string for each backend version. If an environment variable for a given Postgresql major version is missing, testing against that version is skipped.

For example, to test against Postgresql versions 9.3 and 9.1, define NPGSQL_TEST_DB_9.3 and NPGSQL_TEST_DB_9.1, each containing a valid Postgresql connection string.

After that, you have to configure each backend to listen to different tcp ports. Remember to add the port parameter to the connection string you specified above!
