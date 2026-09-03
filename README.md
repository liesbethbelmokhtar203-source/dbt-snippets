# dbt-snippets

The macros and model patterns I copy into every new dbt project. Not a package,
not a framework, just the five or six things I was tired of rewriting.

Tested against dbt-core 1.7 on Postgres and DuckDB. The macros use dbt's
cross-database helpers (`dbt.dateadd`, `dbt.type_numeric`), so they should work
on Snowflake and BigQuery too, but I have not run them there recently.

## Install

Copy what you need into your own project:

```sh
cp macros/utils.sql /path/to/your-project/macros/
```

Or point a `packages.yml` at this repo if you want the macros to update:

```yaml
packages:
  - git: "https://github.com/liesbethbelmokhtar203-source/dbt-snippets.git"
    revision: main
```

## Usage

`limit_dev_rows` keeps development runs small. It emits a `where` clause outside
production and nothing during a full refresh:

```sql
select * from {{ source('shop', 'orders') }}
{{ limit_dev_rows('created_at', 7) }}
```

`safe_divide` returns null instead of erroring on a zero denominator:

```sql
{{ safe_divide('sum(revenue)', 'count(*)') }} as avg_order_value
```

`cents_to_amount` converts integer cents once, in staging, so no mart has to
guess the unit. `generate_schema_name` keeps every dev run in a single schema
instead of scattering models across a dozen of them. `grant_select` goes in a
post-hook so a new model is readable without a manual grant.

The two models under `models/` are the shape I use for staging and for an
incremental daily fact, including a three day lookback for late-arriving rows.
They reference a `shop.orders` source that does not exist here, so read them,
do not run them.

## License

MIT.
