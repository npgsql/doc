---
layout: post
author: roji
title:  "Npgsql 3.1.4 and 3.0.8"
date:   2016-06-15
---
Npgsql 3.1.4 and 3.0.8 have been released and are available on nuget.org.

3.1.4 mainly fixes some more cases of missing `ConfigureAwait(false)` which could cause deadlocks,
you are strongly encouraged to upgrade, especially if you use the async APIs.

3.0.8 is an important update in the 3.0.x line. You should definitely upgrade to 3.1.x, but if for
some reason you can't 3.0.8 fixes quite a few bugs.

Also, I'm away on a pretty intensive 5-month and won't be able to invest as much time in Npgsql as
I could recently. I'll do my best to solve bugs but work on 3.2 probably won't start before November.
Please be patient :)

The complete list of issues [for 3.1.4 is here](https://github.com/npgsql/npgsql/issues?q=milestone%3A3.1.4).
The complete list of issues [for 3.0.8 is here](https://github.com/npgsql/npgsql/issues?q=milestone%3A3.0.8).
