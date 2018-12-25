---
title: "DB schema"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 1
pre: "<i class='fa fa-database'>&nbsp;</i> "
---
##### Create schema script generated by KittyORM for schema `basic_database` v. `1`
{{< highlight sql >}}
CREATE TABLE IF NOT EXISTS cai (id INTEGER NOT NULL PRIMARY KEY, rnd_id INTEGER NOT NULL UNIQUE, animal TEXT CHECK (animal IN ("CAT", "TIGER", "LION")), default_number INTEGER NOT NULL DEFAULT 28, creation_date TEXT NOT NULL DEFAULT  CURRENT_DATE , creation_tmstmp INTEGER NOT NULL DEFAULT  CURRENT_TIMESTAMP , CONSTRAINT CAI_FK FOREIGN KEY(rnd_id) REFERENCES random (id) ON UPDATE CASCADE ON DELETE CASCADE);

CREATE INDEX IF NOT EXISTS basic_database_cai_creation_date_INDEX ON cai (creation_date);

CREATE UNIQUE INDEX IF NOT EXISTS IAC_unique_index_creation_timestamp ON cai (creation_tmstmp);

CREATE TABLE IF NOT EXISTS random (id INTEGER NOT NULL PRIMARY KEY ASC, random_int INTEGER, rnd_int_custom_column_name INTEGER, rndanimal TEXT, random_animal_name TEXT, random_animal_says TEXT);

CREATE INDEX IF NOT EXISTS random_animal_index ON random (rndanimal);

CREATE TABLE IF NOT EXISTS complex_random (id INTEGER NOT NULL PRIMARY KEY ASC, random_int INTEGER, rnd_int_custom_column_name INTEGER, rndanimal TEXT, random_animal_name TEXT, bool_f INTEGER, byte_f INTEGER, double_f REAL, long_f INTEGER, short_f INTEGER, float_f REAL, byte_array NONE, string_f TEXT, big_decimal_f TEXT, big_integer_f TEXT, uri_f TEXT, file_f TEXT, currency_f TEXT, string_s_d_f TEXT, bitmap_colour TEXT, byte_array_s_d_f BLOB, bool_f_f INTEGER, byte_f_f INTEGER, double_f_f REAL, short_f_f INTEGER, float_f_f REAL, long_f_f INTEGER, date_f INTEGER, calendar_f INTEGER, timestamp_f INTEGER);
{{< /highlight >}} 
##### Drop schema script generated by KittyORM for schema `basic_database` v. `1`
{{< highlight sql >}}
DROP TABLE IF EXISTS cai;

DROP TABLE IF EXISTS random;

DROP TABLE IF EXISTS complex_random;
{{< /highlight >}} 