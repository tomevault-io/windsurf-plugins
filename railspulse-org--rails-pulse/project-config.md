---
trigger: always_on
description: Rails Pulse has a three-path migration architecture. Always keep all three paths in mind when making schema changes.
---

# Rails Pulse — Claude Context

## Database Migrations

Rails Pulse has a three-path migration architecture. Always keep all three paths in mind when making schema changes.

### How it works

- **`db/rails_pulse_schema.rb`** is the single source of truth for the complete schema. It is used only for fresh installations — it checks `table_exists?` before creating each table and will never modify existing tables.
- **`lib/generators/rails_pulse/templates/db/rails_pulse_schema.rb`** is the generator template that gets copied into users' apps when they run `rails generate rails_pulse:install`. It must always be kept identical to `db/rails_pulse_schema.rb`.
- **`db/rails_pulse_migrate/`** contains incremental migrations for new features. These are copied into users' apps by the upgrade generator (`rails generate rails_pulse:upgrade`).
- The install generator (`rails generate rails_pulse:install`) copies the template schema into the user's app, then creates a migration that loads and executes it.

### When adding a new column or table

You must update **all three** (plus the table list for new tables):

1. **`db/rails_pulse_schema.rb`** — add the column/table for fresh installs (source of truth)
2. **`lib/generators/rails_pulse/templates/db/rails_pulse_schema.rb`** — keep in sync with the source of truth so the install generator produces correct output
3. **`db/rails_pulse_migrate/TIMESTAMP_description.rb`** — add an incremental migration with `column_exists?`/`table_exists?` guards for existing installs
4. **`lib/generators/rails_pulse/base_methods.rb` `RAILS_PULSE_TABLES`** — add the table name (new tables only); this is the fallback list used when the host app's schema file can't be parsed
5. **`test/migrations/upgrade_migration_test.rb`** — update the migration regression tests:
   - Add the new class name to `MIGRATION_CLASSES` (in filename sort order)
   - Add an assertion that the new column/table exists after upgrading from v0.2.7
   - Run `rake test_migrations` to verify

Example incremental migration pattern:
```ruby
class AddPriorityToJobs < ActiveRecord::Migration[7.0]
  def change
    unless column_exists?(:rails_pulse_jobs, :priority)
      add_column :rails_pulse_jobs, :priority, :integer, default: 0
    end
  end
end
```

### Test/dummy app

The test dummy app needs its schema kept in sync:
- `test/dummy/db/rails_pulse_schema.rb` must mirror `db/rails_pulse_schema.rb`
- Run `rake sync_test_schema` to sync them (also runs automatically before test setup)

### What NOT to do

- Do not modify existing table structure in `db/rails_pulse_schema.rb` — it only creates tables, never alters them
- Do not add columns directly to the schema file without also adding an incremental migration in `db/rails_pulse_migrate/`
- Do not use acronyms in migration filenames (e.g. `sql`, `url`, `id`). Rails camelizes `add_actual_sql_to_operations` to `AddActualSqlToOperations`, but some host apps use inflection rules that produce `AddActualSQLToOperations`, causing a `NameError` at runtime. Use full words instead: `add_actual_query_to_operations`, `add_endpoint_to_routes`, etc.
- **Never use model classes inside `up` migrations for data backfills.** `Model.where(...).update_all(...)` checks out a connection from the model's pool. On separate-database setups with SQLite, the column added earlier in the same migration transaction is invisible to that second pool, causing the migration to roll back. Always use `execute(<<~SQL ... SQL)` for data changes within migrations — `execute` runs on the migration's own connection and sees all DDL performed in the same transaction.

Full architecture details: `docs/database_setup.md`

## Running Tests

```bash
DB=sqlite3 rails test          # Default (SQLite3 + Rails 8.0)
DB=postgresql rails test       # PostgreSQL
rake test_matrix               # All DBs × Rails versions (pre-release validation)
BROWSER=true rake test_matrix  # Include system tests
```

Tests are parallelized by default. System tests (`BROWSER=true`) disable parallelization automatically.

## Testing Conventions

Follow the principles in `docs/testing.md`. Key rules:

- Use **fixtures** as the default for test data (declared at class top with `fixtures :table_names`)
- Test only **public methods** — private methods are tested indirectly
- Never use bare `rescue` in tests; use `assert_raises` instead
- Use `travel_to`/`travel_back` for time-based tests; always clean up in teardown
- Use specific assertions: `assert_operator`, `assert_includes`, `assert_in_delta`, `assert_difference`
- Organize tests with comment headers: `# Structure Tests`, `# Calculation Tests`, `# Edge Cases`
- Always test edge cases: nil, empty, zero, boundary values, 100% scenarios

## Code Style

RuboCop with `rubocop-rails-omakase` enforces style. Run `bundle exec rubocop` before committing.

Naming conventions:
- Classes: `RailsPulse::FeatureName` (always namespaced)
- Models: singular (`Route`, `Request`, `Job`)
- Tables: `rails_pulse_<plural>` (e.g., `rails_pulse_jobs`)
- All models inherit from `RailsPulse::ApplicationRecord`

## Architecture Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [railspulse-org/rails_pulse](https://github.com/railspulse-org/rails_pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
