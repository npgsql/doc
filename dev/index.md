---
layout: page
title: Developer Resources
---

## Tests

We maintain a large regression test suite, if you're planning to submit code, please provide a test
that reproduces the bug or tests your new feature. See [this page](tests.html) for information on the
Npgsql test suite.

## Build Server

We have a [TeamCity build server](https://www.jetbrains.com/teamcity/) running continuous integration builds
on commits pushed to our github repository. The Npgsql testsuite is executed over all officially supported
PostgreSQL versions to catch errors as early as possible. CI NuGet packages are automatically pushed to our
[unstable feed at MyGet](https://www.myget.org/F/npgsql-unstable).

For some information about the build server setup, see [this page](build-server.html).

Thanks to Dave Page at PostgreSQL for donating a VM for this!

## Release Checklist

These are the steps needed to publish release 3.0.6:

* Merge --no-ff hotfix/3.0.6 into master
* Tag master with v3.0.6
* Push both master and v3.0.6 to Github
* Wait for the build to complete
* In TeamCity, go to the artifacts for the build and download them all as a single ZIP
* Nuget push the packages
* Write release notes on npgsql.org, publish
* Create release on github, pointing to npgsql.org
* Upload MSI to the github release
* Delete hotfix/3.0.6 both locally and on github
* Create new branch hotfix/3.0.7 off of master, push to github

## Other stuff

Emil compiled [a list of PostgreSQL types and their wire representations](types.html).

