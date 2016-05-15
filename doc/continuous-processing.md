---
layout: doc
title: Continuous Processing
---

Note: *Continuous processing has been removed in Npgsql 3.1, and [replaced by `Wait`](wait.html)*.

## PostgreSQL Asynchronous messages

PostgreSQL has a feature whereby arbitrary notification messages can be sent between clients. For example, one client may wait until it is
notified by another client of a task that it is supposed to perform. Notifications are, by their nature, asynchronous - they can arrive
at any point. For more detail about this feature, see the PostgreSQL [NOTIFY command](http://www.postgresql.org/docs/current/static/sql-notify.html).
Some other asynchronous message types are notices (e.g. database shutdown imminent) and parameter changes, see the
[PostgreSQL protocol docs](http://www.postgresql.org/docs/current/static/protocol-flow.html#PROTOCOL-ASYNC) for more details.

Note that despite the word "asynchronous", this page has nothing to do with ADO.NET async operations.

---

## Processing of Asynchronous Messages

Npgsql exposes notification messages via the Notification event on NpgsqlConnection.

Since asynchronous notifications are rarely used and processing them is complex, by default Npgsql only processes notification messages as
part of regular (synchronous) query interaction. That is, if an asynchronous notification is sent, Npgsql will only process it and emit an
event to the user the next time a command is sent and processed. To make Npgsql process messages at any time, even if no query is in
progress, enable the [Continuous Processing](connection-string-parameters.html#continuous-processing) flag in your connection string. 

{% highlight C# %}

var conn = new NpgsqlConnection(ConnectionString + ";ContinuousProcessing=true");
conn.Open();
conn.Notification += (o, e) => Console.WriteLine("Received notification");

using (var cmd = new NpgsqlCommand("LISTEN notifytest", conn))
{
    cmd.ExecuteNonQuery();
{

// From this point your connection will fire the Notification event whenever a PostgreSQL
// notification arrives. Keep the connection open and referenced to make sure it doesn't go 
// out of scope and garbage collected.

{% endhighlight %}

---

## Keepalive

You may want to turn on [keepalives](keepalive.html).
