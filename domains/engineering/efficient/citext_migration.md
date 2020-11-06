---
title: Migrating email field to citext
nav_order: 15
parent: Efficient
grand_parent: Engineering
---


# Migrating email field from string to citext 

### What is citext?

Postgres provides with a column extension called **CITEXT** (Case-Insensitive Extension). The entension allows us to create fields of type citext in postgresql. As the name suggests, it allows us to query fields of type citext in a case insensitive manner.

Assuming a table exists named `User` with a field `name` of type `citext`

```
User.where(name: "Test")
User.where(name: "test")
```
Both the above queries will return the same output. The reason for this is, Postgres does a case insensitive comparision of CITEXT columns by converting the query string and the value of the comparing column to lowercase using LOWER.   

**Note:** 
- The data on the database is not exactly stored in lowercase even though it may seem so.
- Querying CITEXT column is slower than a TEXT or STRING column because of the underlying lowercase conversion.

### Why have email field as citext type?

Emails are case insensitive and thus should be treated the same way. Ideally when someone provides an email, for example `Test@example.com` or `test@example.com`, they shouldn't be differentiated. They need to be considered the same and querying through them should eventually lead to the same result.

With the citext extension in PostgreSQL, any comparisons on a column of citext type will internally have the lower function executed on the arguments. Thus, its ideal to have email fields of citext type.

### Migrating Email column to citext

To add or change a column to citext, we must first enable the citext extension using the following command in a migration:
```
def change
  enable_extension 'citext'
end
```
After enabling the citext extension, create migrations as follows:  
To add a citext column in a new table:
```
def change
  create_table :MODEL do |t|
    t.citext :COLUMN_NAME

    t.timestamps
  end
end
```
To add a new column/change an existing column to citext type:
```
def change
  add_column :MODEL, :COLUMN_NAME, :citext
  change_column :MODEL, :COLUMN_NAME, :citext
end
```
FInally, run `rails db:migrate`