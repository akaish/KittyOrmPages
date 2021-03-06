---
title: "DB schema"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 1
pre: "<i class='fa fa-database'>&nbsp;</i> "
---
##### Create schema script generated by KittyORM for database `mig` version `3`
{{< highlight sql >}}
CREATE TABLE IF NOT EXISTS mig_three (id INTEGER NOT NULL PRIMARY KEY ASC, some_value TEXT NOT NULL DEFAULT 'Something random');

CREATE TABLE IF NOT EXISTS mig_one (id INTEGER NOT NULL PRIMARY KEY ASC, creation_date TEXT NOT NULL DEFAULT  CURRENT_DATE , default_integer INTEGER DEFAULT 228);

CREATE INDEX IF NOT EXISTS m1_di_index ON mig_one (default_integer);

CREATE TABLE IF NOT EXISTS mig_two (id INTEGER NOT NULL PRIMARY KEY ASC, mig_one_reference INTEGER REFERENCES mig_one (id) ON UPDATE CASCADE ON DELETE CASCADE, some_animal TEXT, some_animal_sound TEXT);
{{< /highlight >}} 
##### Drop schema script generated by KittyORM for database `mig` version `3`
{{< highlight sql >}}
DROP TABLE IF EXISTS mig_one;

DROP TABLE IF EXISTS mig_two;

DROP TABLE IF EXISTS mig_three;
{{< /highlight >}} 
##### Migration script generated by KittyORM for database `mig` from version `2` to version `3` (SimpleMigrationScriptGenerator migrator)
{{< highlight sql >}}
ALTER TABLE mig_one RENAME TO mig_one_t_old;

CREATE TABLE IF NOT EXISTS mig_one (id INTEGER NOT NULL PRIMARY KEY ASC, creation_date TEXT NOT NULL DEFAULT  CURRENT_DATE , default_integer INTEGER DEFAULT 228);

INSERT INTO mig_one (id, creation_date, default_integer) SELECT id, creation_date, default_integer FROM mig_one_t_old;

DROP TABLE IF EXISTS mig_one_t_old;

CREATE INDEX IF NOT EXISTS m1_di_index ON mig_one (default_integer);

ALTER TABLE mig_two ADD COLUMN some_animal_sound;

CREATE TABLE IF NOT EXISTS mig_three (id INTEGER NOT NULL PRIMARY KEY ASC, some_value TEXT NOT NULL DEFAULT 'Something random');

DROP INDEX IF EXISTS mig.m2_sa_index;
{{< /highlight >}} 
##### Migration script generated by KittyORM for database `mig` from version `1` to version `3` (SimpleMigrationScriptGenerator migrator)
{{< highlight sql >}}
ALTER TABLE mig_one RENAME TO mig_one_t_old;

CREATE TABLE IF NOT EXISTS mig_one (id INTEGER NOT NULL PRIMARY KEY ASC, creation_date TEXT NOT NULL DEFAULT  CURRENT_DATE , default_integer INTEGER DEFAULT 228);

INSERT INTO mig_one (id, creation_date) SELECT id, creation_date FROM mig_one_t_old;

DROP TABLE IF EXISTS mig_one_t_old;

CREATE INDEX IF NOT EXISTS m1_di_index ON mig_one (default_integer);

CREATE TABLE IF NOT EXISTS mig_two (id INTEGER NOT NULL PRIMARY KEY ASC, mig_one_reference INTEGER REFERENCES mig_one (id) ON UPDATE CASCADE ON DELETE CASCADE, some_animal TEXT, some_animal_sound TEXT);

CREATE TABLE IF NOT EXISTS mig_three (id INTEGER NOT NULL PRIMARY KEY ASC, some_value TEXT NOT NULL DEFAULT 'Something random');
{{< /highlight >}} 
