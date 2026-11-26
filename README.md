This is the documentation repo for Npgsql.

It contains the site shell and the conceptual documentation for EntityFramework6.Npgsql (AKA EF6.PG). The conceptual documentation for Npgsql and Npgsql.EntityFrameworkCore.PostgreSQL (AKA EFCore.PG) lives in the `docs` directory of their respective repos ([npgsql/npgsql](https://github.com/npgsql/npgsql), [npgsql/efcore.pg](https://github.com/npgsql/efcore.pg)).

To build locally, clone the Npgsql and EFCore.PG repos into `Npgsql` and `EFCore.PG` in the repo root - docfx takes the conceptual docs from their `docs` directories, and extracts API documentation from their sources.

A GitHub Actions workflow clones the latest `main` of both repos, rebuilds the entire documentation and pushes the results to live. It runs on pushes to this repo, when the Npgsql or EFCore.PG repos signal a docs change (via `repository_dispatch`), and nightly.
