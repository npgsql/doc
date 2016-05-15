---
layout: doc
title: Migration Notes
---

## Migrating from 3.0 to 3.1

* CommandTimeout used to be implemented with PostgreSQL's `statement_timeout` parameter, but this wasn't a very reliable
  method and has been removed. CommandTimeout is now implemented via socket timeouts only, see
  [#689](https://github.com/npgsql/npgsql/issues/689) for more details.
  Note that if a socket timeout occurs, the connection is broken and must be reopened.
* The [`Persist Security Info`](connection-string-parameters.html#persist-security-info) parameter has been implemented
  and is false by default. This means that once a connection has been opened, you will not be able to get its password.
* Removed ContinuousProcessing mode, and replaced it with [Wait](wait.html), a simpler and less bug-prone
  mechanism for consuming asynchronous notifications ([#1024](https://github.com/npgsql/npgsql/issues/1024)).
* The `Maximum Pool Size` connection is parameter is now 100 default instead of 20 (this is default in SqlClient, pg_bouncer...).
* The `Connection Lifetime` parameter has been renamed to `Connection Idle Lifetime`, and its default has been changed from
  15 to 300. Also, once the number of seconds has elapsed the connection is closed immediately; the previous behavior closed
  half of the connections.
* `RegisterEnum` and `RegisterEnumGlobally` have been renamed to `MapEnum` and `MapEnumGlobally` respectively.
* If you used enum mapping in 3.0, the strategy for translating between CLR and PostgreSQL type names has changed. In 3.0
  Npgsql simply used the CLR name (e.g. SomeField) as the PostgreSQL name; Npgsql 3.1 uses a user-definable name translator,
  default to snake case (e.g. some_field). See [#859](https://github.com/npgsql/npgsql/issues/859).
* The `EnumLabel` attribute has been replaced by the `PgName` attribute (which is also used for the new composite type support).
* When PostgreSQL sends an error, it is no longer raised by an NpgsqlException but by a PostgresException.
  PostgresException is a subclass of NpgsqlException so code catching NpgsqlException should still work, but the
  PostgreSQL-specific exception properties will only be available on PostgresException.
* The Code property on NpgsqlException has been renamed to SqlState. It has also been moved to PostgresException.
* NpgsqlNotice has been renamed to PostgresNotice
* For multistatement commands, PostgreSQL parse errors will now be thrown only when the user calls NextResult() and gets to
  the problematic statement.
* It is no longer possible to dispose a prepared statement while a reader is still open.
  Since disposing a prepared statement includes database interaction, the connection must be idle.
* Removed `NpgsqlConnection.SupportsHexByteFormat`.
* Renamed `NpgsqlConnection.Supports_E_StringPrefix` to `SupportsEStringPrefix`.

## Migrating from 2.2 to 3.0

Version 3.0 represents a near-total rewrite of Npgsql. In addition to changing how Npgsql works internally and communicates
with PostgreSQL, a conscious effort was made to better align Npgsql with the ADO.NET specs/standard and with SqlClient
where that made sense. This means that you cannot expect to drop 3.0 as a replacement to 2.2 and expect things to work
- upgrade cautiously and test extensively before deploying anything to production.

The following is a *non-exhaustive* list of things that changed. If you run against a breaking change not documented here,
please let us know and we'll add it.

### Major

* Support for .NET 2.0, .NET 3.5 and .NET 4.0 has been dropped - you will have to upgrade to .NET 4.5 to use Npgsql 3.0.
  We'll continue to do bugfixes on the 2.2 branch for a while on a best-effort basis.
* The Entity Framework provider packages have been renamed to align with Microsoft's new naming.
  The new packages are *EntityFramework5.Npgsql* and *EntityFramework6.Npgsql*. EntityFramework7.Npgsql is in alpha.
* A brand-new bulk copy API has been written, using binary encoding for much better performance.
  See [the docs](copy.html).
* Composite (custom) types aren't supported yet, but this is a high-priority feature for us.
  See [#441](https://github.com/npgsql/npgsql/issues/441).

### SSL

* Npgsql 2.2 didn't perform validation on the server's certificate by default, so self-signed certificate were accepted.
  The new default is to perform validation. Specify the
  [Trust Server Certificate](connection-string-parameters.html#trust-server-certificate) connection string parameter to get back previous behavior.
* The "SSL" connection string parameter has been removed, use "SSL Mode" instead.
* The "SSL Mode" parameter's Allow option has been removed, as it wasn't doing anything.

### Type Handling

* Previously, Npgsql allowed writing a NULL by setting NpgsqlParameter.Value to `null`.
  This is [not allowed in ADO.NET](https://msdn.microsoft.com/en-us/library/system.data.common.dbparameter.value%28v=vs.110%29.aspx)
  and is no longer supported, set to `DBNull.Value` instead.
* In some cases, you will now be required to explicitly set a parameter's type although you didn't have to before (you'll get an error
  42804 explaining this). This can happen especially in Dapper custom custom type handlers ([#694](https://github.com/npgsql/npgsql/issues/694)).
  Simply set the NpgsqlDbType property on the parameter.
* Removed support for writing a parameter with an `IEnumerable<T>` value, since that would require Npgsql to enumerate it multiple
  times internally. `IList<T>` and IList are permitted.
* It is no longer possible to write a .NET enum to an integral PostgreSQL column (e.g. int4).
  Proper enum support has been added which allows writing to PostgreSQL enum columns
  (see [the docs](http://www.npgsql.org/doc/enum.html).
  To continue writing enums to integral columns as before, simply add an explicit cast to the integral type in your code.
* NpgsqlMacAddress has been removed and replaced by the standard .NET PhysicalAddress.
* Npgsql's BitString has been removed and replaced by the standard .NET BitArray.
* NpgsqlTime has been removed and replaced by the standard .NET TimeSpan.
* NpgsqlTimeZone has been removed.
* NpgsqlTimeTZ now holds 2 TimeSpans, rather than an NpgsqlTime and an NpgsqlTimeZone.
* NpgsqlTimeStamp no longer maps DateTime.{Max,Min}Value to {positive,negative}
  infinity. Use NpgsqlTimeStamp.Infinity and NpgsqlTimeStamp.MinusInfinity explicitly for that.
  You can also specify the "Convert Infinity DateTime" connection string parameter to retain the old behavior.
* Renamed NpgsqlInet's addr and mask to Address and Mask.
* NpgsqlPoint now holds Doubles instead of Singles ([#437](https://github.com/npgsql/npgsql/issues/437)).
* NpgsqlDataReader.GetFieldType() and GetProviderSpecificFieldType() now return Array for arrays.
  Previously they returned int[], even for multidimensional arrays.
* NpgsqlDataReader.GetDataTypeName() now returns the name of the PostgreSQL type rather than its OID.

### Retired features

* Removed the "Preload Reader" feature, which loaded the entire resultset into memory. If you require this
  (inefficient) behavior, read the result into memory outside Npgsql. We plan on working on MARS support,
  see [#462](https://github.com/npgsql/npgsql/issues/462).
* The "Use Extended Types" parameter is no longer needed and isn't supported. To access PostgreSQL values
  that can't be represented by the standard CLR types, use the standard ADO.NET
  `NpgsqlDataReader.GetProviderSpecificValue` or even better, the generic
  `NpgsqlDataReader.GetFieldValue<T>`.
* Removed the feature where Npgsql automatically "dereferenced" a resultset of refcursors into multiple
  resultsets (this was used to emulate returning multiple resultsets from stored procedures). Note that if
  your function needs to return a single resultset, it should be simply returning a table rather than a
  cursor (see `RETURNS TABLE`).
  See [#438](https://github.com/npgsql/npgsql/issues/438).
* Removed the AlwaysPrepare connection string parameter
* Removed the Encoding connection string parameter, which was obsolete and unused anyway
* Removed the Protocol connection string parameter, which was obsolete and unused anyway (protocol 3 was always used)
  (UTF8 was always used regardless of what was specified)
* Removed NpgsqlDataReader.LastInsertedOID, it did not allow accessing individual OIDs in multi-statement commands.
  Replaced with NpgsqlDataReader.Statements, which provides OID and affected row information on a statement-by-statement
  basis.
* Removed `NpgsqlDataReader.HasOrdinal`, was a badly-named non-standard API without a serious use case.
  `GetName()` can be used as a workaround.

### Other

* It is no longer possible to create database entities (tables, functions) and then use them in the same multi-query command -
  you must first send a command creating the entity, and only then send commands using it.
  See [#641](https://github.com/npgsql/npgsql/issues/641) for more details.
* Previously, Npgsql set DateStyle=ISO, lc_monetary=C and extra_float_digits=3 on all connections it created. This is no longer
  case, if you rely on these parameters you must send them yourself.
* NpgsqlConnection.Clone() will now only return a new connection with the same connection string as the original.
  Previous versions returned an open connection if the original was open, and copied the Notice event listeners as well.
  Note: NpgsqlConnection.Clone() was accidentally missing from 3.0.0 and 3.0.1.
* Removed the obsolete `NpgsqlParameterCollection.Add(name, value)` method. Use `AddWithValue()` instead, which also exists
  in SqlClient.
* <del>The savepoint manipulation methods on `NpgsqlTransaction` have been renamed from `Save`, and `Rollback` to
  `CreateSavepoint` and `RollbackToSavepoint`.</del>
  This broke the naming conventions for these methods across other providers (SqlClient, Oracle...) and so in 3.0.2 the previous
  names were returned and the new names marked as obsolete.
  3.1 will remove the the new names and leaves only `Save` and `Rollback`. See [#738](https://github.com/npgsql/npgsql/issues/738).
* The default CommandTimeout has changed from 20 seconds to 30 seconds, as in
  [ADO.NET](https://msdn.microsoft.com/en-us/library/system.data.idbcommand.commandtimeout(v=vs.110).aspx).
* `CommandType.TableDirect` now requires CommandText to contain the name of a table, as per the
  [MSDN docs](https://msdn.microsoft.com/en-us/library/system.data.commandtype%28v=vs.110%29.aspx).
  Multiple tables (join) aren't supported.
* `CommandType.StoredProcedure` now requires CommandText contain *only* the name of a function, without parentheses or parameter
  information, as per the [MSDN docs](https://msdn.microsoft.com/en-us/library/system.data.commandtype%28v=vs.110%29.aspx).
* Moved the `LastInsertedOID` property from NpgsqlCommand to NpgsqlReader, like the standard ADO.NET `RecordsAffected`.
