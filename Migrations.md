# Migrations
- Each table should have its own migration file.
- Migrations should only change the database schema, and not manipulate data, unless directly necessary for the migration to run.
- Migrations should contain the definitive table definition in the up method, and only drop the table in the down method.
- Migrations should not be used to alter database structures, as this will prevent testing using in-memory SQLite databases. (See Seeders for this.)

## Primary Keys
- All primary keys should be defined as `$table->id();`.

## Foreign Keys
- Foreign keys should be set on all fields that refer to another table.
- Foreign key fields should be defined as `$table->unsignedBigInteger()`.
- Try to avoid manually naming the FK, instead let Laravel generate the name. This ensures consistent and predicatable naming across the board.

### onDelete()
- Use `CASCADE` when the current table record should be deleted if the refering record is deleted.
- Use `SET NULL` when the current table record should be preserved, even if the refering record is deleted. If the foreign key field is nullable you should use `SET NULL`.
- Use `RESTRICT` when the deletion of the refering record should be prevented if the current record exists.

### onUpdate()
- This should almost always be set to `CASCADE`, meaning if the primary key of the refering record changes, this foreign key will update to retain the relationship.
