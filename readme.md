## Drizzle Kit
DrizzleKit - is a CLI migrator tool for DrizzleORM. It is probably one and only tool that lets you completely automatically generate SQL migrations and covers ~95% of the common cases like delitions and renames by prompting user input.

### How it works
`drizzle-kit` will traverse `data folder` from configuration file, find all schema .ts files. Generate schema snapshot and compare it to the previous version(if there's one). Based on the difference it will generate all needed SQL migrations and if there're any `automatically unresolvable` cases like `renames` it will prompt user for input.

For schema file:
```typescript
import { AbstractTable } from "drizzle-orm";

export class UsersTable extends AbstractTable<UsersTable> {
  public id = this.serial("id").primaryKey();
  public fullName = this.varchar("full_name", { size: 256 });

  public fullNameIndex = this.index(this.fullName);

  public tableName(): string {
    return "users";
  }
}

export class AuthOtpTable extends AbstractTable<AuthOtpTable> {
  public id = this.serial("id").primaryKey();
  public phone = this.varchar("phone", { size: 256 });
  public userId = this.int("user_id").foreignKey(UsersTable, (t) => t.id);

  public tableName(): string {
    return "auth_otp";
  }
}
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
```bash
npm install -g drizzle-kit
```
Create a `drizzle.config.yml` configuration file:
```yaml
migrationRootFolder: drizzle ## all migrations will live here
dataFolder: './src/data'     ## where are all schema .ts files
```
  \
That's it, you're ready to go 🚀
```
> drizzle-kit migrate
```
  \
You can also run migrations in project scope
```js
// package.json
{
  ...
  scripts: {
    ...
    migrate: "drizzle-kit migrate"
  }
}

> npm run migrate
```
    

