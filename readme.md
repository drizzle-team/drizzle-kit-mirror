## Drizzle Kit

DrizzleKit - is a CLI migrator tool for DrizzleORM. It is probably one and only tool that lets you completely automatically generate SQL migrations and covers ~95% of the common cases like deletions and renames by prompting user input.
<https://github.com/drizzle-team/drizzle-kit-mirror> - is a mirror repository for issues.

## Documentation

Check the full documenation on [the website](https://orm.drizzle.team/kit-docs/overview)

### How it works

`drizzle-kit` will traverse `schema folder` or `schema file`, generate schema snapshot and compare it to the previous version, if there's one.
 Based on the difference it will generate all needed SQL migrations and if there are any `automatically unresolvable` cases like `renames` it will prompt user for input.

For schema file:

```typescript
// ./src/db/schema.ts

import { integer, pgTable, serial, text, varchar } from "drizzle-orm/pg-core";

const users = pgTable("users", {
    id: serial("id").primaryKey(),
    fullName: varchar("full_name", { length: 256 }),
  }, (table) => ({
    nameIdx: index("name_idx", table.fullName),
  })
);

export const authOtp = pgTable("auth_otp", {
  id: serial("id").primaryKey(),
  phone: varchar("phone", { length: 256 }),
  userId: integer("user_id").references(() => users.id),
});
```

It will generate:

```SQL
CREATE TABLE IF NOT EXISTS auth_otp (
 "id" SERIAL PRIMARY KEY,
 "phone" character varying(256),
 "user_id" INT
);

CREATE TABLE IF NOT EXISTS users (
 "id" SERIAL PRIMARY KEY,
 "full_name" character varying(256)
);

DO $$ BEGIN
 ALTER TABLE auth_otp ADD CONSTRAINT auth_otp_user_id_fkey FOREIGN KEY ("user_id") REFERENCES users(id);
EXCEPTION
 WHEN duplicate_object THEN null;
END $$;

CREATE INDEX IF NOT EXISTS users_full_name_index ON users (full_name);
```

### Installation & configuration

```shell
npm install -D drizzle-kit
```

Running with CLI options

```jsonc
// package.json
{
 "scripts": {
  "generate": "drizzle-kit generate:pg --out migrations-folder --schema src/db/schema.ts"
 }
}
```

```shell
npm run generate
```

### Configuration

Instead of passing arguments to the CLI commands, you can also create a `drizzle.config.json` configuration file:

```json
{
  "out": "./migrations-folder",
  "schema": "./src/db",
  "breakpoints": false
}
```

* `out` - path to the output directory where migrations will be saved,
* `schema` - path to TypeScript schema file or folder with multiple schema files,
* `breakpoints` - whether to execute each statement in the migrations individually. Set to `true` if you are using SQLite or MySQL (as they don't support multiple DDL statements in a transaction).

---

## Upgrading to 0.17.0

Before running any new migrations `drizzle-kit` will ask you to upgrade first

Migration file structure < 0.17.0

```plaintext
📦 <project root>
 └ 📂 migrations
    └ 📂 20221207174503
       ├ 📜 migration.sql
       ├ 📜 snapshot.json
    └ 📂 20230101104503
       ├ 📜 migration.sql
       ├ 📜 snapshot.json
```

Migration file structure >= 0.17.0

```plaintext
📦 <project root>
 └ 📂 migrations
    └ 📂 meta
      ├ 📜 _journal.json
      ├ 📜 0000_snapshot.json
      ├ 📜 0001_snapshot.json
    └ 📜 0000_icy_stranger.sql
    └ 📜 0001_strange_avengers.sql
```

To easily migrate from previous folder structure to new you need to run `up` command in drizzle-kit. It's a great helper to upgrade your migrations to new format on each drizzle kit major update

![](media/up_mysql.gif)
---

## List of commands

### Generate SQL migrations based on current .ts schema

---

**`$ drizzle-kit generate:pg`** \
**`$ drizzle-kit generate:mysql`** \
**`$ drizzle-kit generate:sqlite`**

`--config` [optional default=drizzle.config.json] config file path\
`--schema` path to typescript schema file or folder with multiple schema files\
`--out` [optional default=drizzle/] migrations folder\
`--breakpoints` [optional default=false] whether to execute each statement in the migrations individually\

```shell
$ drizzle-kit generate:pg
## runs generate command with drizzle.config.json

$ drizzle-kit generate:pg --config=./custom.config.json
## runs generate command with custom.config.json

$ drizzle-kit generate:pg --schema=./src/schema.ts
## runs generate command and outputs results to ./drizzle

$ drizzle-kit generate:pg --schema=./src/schema.ts --out=./migrations/
## runs generate command and outputs results to ./migration
```

**Generate an empty SQL migration with `--custom`**

```shell
$ drizzle-kit generate:pg --custom
## runs generate command and outputs an empty migration for custom SQL
```

### Introspect existing database and generate typescript schema

---

**`$ drizzle-kit introspect:pg`** \
**`$ drizzle-kit introspect:mysql`**

```shell
drizzle-kit introspect:pg --out=migrations/ --connectionString=postgresql://user:pass@host:port/db_name

drizzle-kit introspect:pg --out=migrations/ --host=0.0.0.0 --port=5432 --user=postgres --password=pass --database=db_name --ssl
```

![](media/introspect_mysql.gif)

### Update stale snapshots

---

**`$ drizzle-kit up:pg`** \
**`$ drizzle-kit up:mysql`** \
**`$ drizzle-kit up:sqlite`**

`--out` [optional] migrations folder\
`--config` [optional default=drizzle.config.json] config file path

```shell
## migrations folder is taken from drizzle.config.json
drizzle-kit up:mysql

drizzle-kit up:mysql --out=migrations/
```

![](media/up_mysql.gif)

### Drop migration

---

**`$ drizzle-kit drop`**

`--out` [optional] migrations folder\
`--config` [optional default=drizzle.config.json] config file path

![](media/drop.gif)

### Migrations collisions check

---

**`$ drizzle-kit check:pg`** \
**`$ drizzle-kit check:mysql`** \
**`$ drizzle-kit check:sqlite`**

`--out` [optional] migration folder\
`--config` [optional default=drizzle.config.json] config file path

```shell
## migrations folder is taken from drizzle.config.json
drizzle-kit check:pg

drizzle-kit check:pg --out=migrations/
```
