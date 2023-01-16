## Drizzle Kit
DrizzleKit - is a CLI migrator tool for DrizzleORM. It is probably one and only tool that lets you completely automatically generate SQL migrations and covers ~95% of the common cases like deletions and renames by prompting user input.\
https://github.com/drizzle-team/drizzle-kit-mirror - is a mirror repository for issues.

### How it works
`drizzle-kit` will traverse `schema folder` or `schema file`, generate schema snapshot and compare it to the previous version (if there's one).\
 Based on the difference it will generate all needed SQL migrations and if there are any `automatically unresolvable` cases like `renames` it will prompt user for input.

For schema file:
```typescript
// ./src/db/schema.ts

import { integer, pgTable, serial, text, varchar } from "drizzle-orm-pg";

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
$ npm install -D drizzle-kit
```

Running with CLI options
```shell
$ npm exec drizzle-kit generate --out migrations-folder --dialect pg --schema src/db/schema.ts
```

Or put your file to `drizzle.config.json` configuration file:
```json
{
  "dialect": "pg",
  "out": "./migrations-folder",
  "schema": "./src/db"
}
```
---
### List of commands
**`$ drizzle-kit generate`** - generates SQL migrations based on .ts schema\
`--config` [optional default=drizzle.config.json] path to an optional config file\
`--dialect` [optional default=pg] database dialect, one of -> pg, mysql, sqlite\
`--schema` path to a schema file or folder with multiple schema files\
`--out` [optional default=drizzle/] place where to store migration files\

```shell
$ drizzle-kit generate 
## runs generate command with drizzle.config.json 

$ drizzle-kit generate --config custom.config.json
## runs generate command with custom.config.json 

$ drizzle-kit generate --dialect pg --schema src/schema.ts
## runs generate command and outputs results to ./drizzle

$ drizzle-kit generate --dialect .. --schema .. --out ./migration-folder
## runs generate command and outputs results to ./migration-folder
```  


**`$ drizzle-kit introspect:pg`** - generate `schema.ts` file from existing PG database within seconds
```shell
drizzle-kit introspect:pg --out=migrations/ --connectionString=postgresql://user:pass@host:port/db_name

drizzle-kit introspect:pg --out=migrations/ --host=0.0.0.0 --port=5432 --user=postgres --password=pass --database=db_name --ssl
```

**`$ drizzle-kit up`** - updates stale snapshots
`--config` [optional default=drizzle.config.json] path to an optional config file\
`--dialect` [optional default=pg] database dialect, one of -> pg, mysql, sqlite\
`--schema` path to a schema file or folder with multiple schema files\

**`$ drizzle-kit check`** - checks for collisions
`--config` [optional default=drizzle.config.json] path to an optional config file\
`--dialect` [optional default=pg] database dialect, one of -> pg, mysql, sqlite\
`--schema` path to a schema file or folder with multiple schema files\



