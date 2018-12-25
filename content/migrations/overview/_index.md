---
title: "Migrations overview"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 1
---
{{% panel footer="Demo application screenshots for this article" %}}
<center>![alt text](src/1.png?height=150px&classes=border,inline)  ![alt text](src/2.png?height=150px&classes=border,inline)</center>
{{% /panel %}}
### Migrations mechanism between different schema versions
By design KittyORM has three main mechanisms for supporting migrations between different versions of database. Those utilities called `Migrators` and implements `KittyORMVersionMigrator` abstract class. By default, when your application needs to update database schema KittyORM would just wipe old version database and create new schema, however it is not suitable for production purposes. You can define what migration utility to use by annotating your `KittyDatabase` class implementation with `@KITTY_DATABASE_HELPER` annotation and set its `onUpgradeBehavior` from any option available at `@KITTY_DATABASE_HELPER.UpgradeBehavior` enumeration.  Here is a list of three already implemented by KittyORM database version migrations mechanisms:

1. **DropCreate Migrator** - basic database version migration utility, it creates simple migration script that drops all tables that present in newer schema version and recreates them. Using this migration mechanism is useful for development purposes when database filled with test data and there is no need to save it. Implemented by `KittyDevDropCreateMigrator.class`. It is default `onUpgrade` behavior, if you wish to define it manually, set `@KITTY_DATABASE_HELPER.onUpgradeBehavior` to `KITTY_DATABASE_HELPER.UpgradeBehavior.DROP_AND_CREATE`.

2. **SimpleMigrationScriptGenerator Migrator** - migration utility that tries to generate migration script based on differences between current and new schema and save as many data as possible. See `KittySimpleMigrator.class` for more info. To set this behavior just set `onUpgradeBehavior` property value of `@KITTY_DATABASE_HELPER` to `KITTY_DATABASE_HELPER.UpgradeBehavior.USE_SIMPLE_MIGRATIONS`.

3. **Filescript Migrator** - migration utility that run a sequence of migration scripts stored at file system or at assets. Checks set of files named on one pattern and run SQLite scripts stored in at defined path if such migration sequence is applicable for new schema version. Implemented by `KittyORMVersionFileDumpMigrator.class`. To set this behavior you have to set `onUpgradeBehavior` property value of `@KITTY_DATABASE_HELPER` to `KITTY_DATABASE_HELPER.UpgradeBehavior.USE_FILE_MIGRATIONS`.

Also, you are able to implement your own migration mechanism by extending `KittyDatabaseHelper.class` and setting `onUpgradeBehavior` annotation propertie of `@KITTY_DATABASE_HELPER` that annotates your `KittyDatabase` implementation to `KITTY_DATABASE_HELPER.UpgradeBehavior.USE_CUSTOM_MIGRATOR`.

### Initial database setup
In this lesson we would work with database with name `mig`. In this tab we would create first version of `mig` schema, first iteration consists only from one table `mig.mig_one`. Tap "CREATE MIG..." button to create schema and fill it with some random values (see [KittyORM Demo](/hidden/android "KittyORM Demo at Google Play Market")).

**Mig v.1**

**MigOneModel (mig_one)**

Java type | Name | SQLite name | Constraints
---|---|---|---
`Long` | id | id | PRIMARY KEY
`String` | creationDate | creation_date | NOT_NULL
`Integer` | someInteger | some_integer | -


**Create schema script generated by KittyORM for database `mig` version `1`**
{{< highlight sql >}}
CREATE TABLE IF NOT EXISTS mig_one (id INTEGER NOT NULL PRIMARY KEY ASC, creation_date TEXT NOT NULL, some_integer INTEGER);
{{< /highlight >}} 
**Drop schema script generated by KittyORM for database `mig` version `1`**
{{< highlight sql >}}
DROP TABLE IF EXISTS mig_one;
{{< /highlight >}} 

