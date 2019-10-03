This is the documentation repo for Npgsql.

It contains conceptual documentation articles for Npgsql, Npgsql.EntityFrameworkCore.PostgreSQL (AKA EFCore.PG) and EntityFramework6.Npgsql (AKA EF6.PG).

Note that to properly work, docfx expects to also find the Npgsql and EFCore.PG repos cloned in the repo root - it extracts API documentation from them.

A Github Actions workflow automatically clones the appropriate repository, rebuilds the entire documentation and pushes the results to live.
