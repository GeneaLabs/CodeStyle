# Migrations
## Primary Keys
- All primary keys should be defined as `$table->id();`.

## Foreign Keys
- Foreign keys should be set on all fields that refer to another table.
- Foreign key fields should be defined as `unsignedBigInteger()`.

### onDelete()
- Use `CASCADE` when the current table record should be deleted if the refering record is deleted.
- Use `SET NULL` when the current table record should be preserved, even if the refering record is deleted.
- Use `RESTRICT` when the deletion of the refering record should be prevented if the current record exists.

### onUpdate()
- This should almost always be set to `CASCADE`.
