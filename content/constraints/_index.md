---
title: "Constraints and indexes declarations"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 70
---
{{% panel footer="Demo application screenshots for this article" %}}
<center>![alt text](src/1.png?height=150px&classes=border,inline)  ![alt text](src/2.png?height=150px&classes=border,inline) ![alt text](src/3.png?height=150px&classes=border,inline) ![alt text](src/4.png?height=150px&classes=border,inline) ![alt text](src/5.png?height=150px&classes=border,inline) ![alt text](src/6.png?height=150px&classes=border,inline)</center>
{{% /panel %}}

KittyORM supports indexes and constraints declaration by using annotations. Right now KittyORM offers you functionality to define seven supported by SQLite constraints without any need to define them in raw SQL code. Also in such way you can define table indexes. KittyORM is capable to define database schema with nearly all features that are supported by SQLite ([SQL As Understood By SQLite](https://sqlite.org/lang_createtable.html)) with usage only of annotations. In demo you can play with form for creating and inserting entity that declares all constraints and index.

### Table of contents
1. [`NOT NULL` constraint declaration](#not-null-constraint-declaration)
2. [`DEFAULT` constraint declaration](#default-constraint-declaration)
3. [`UNIQUE` constraint declaration](#unique-constraint-declaration)
4. [`CHECK` constraint declaration](#check-constraint-declaration)
5. [`COLLATE` constraint declaration](#collate-constraint-declaration)
6. [`PRIMARY KEY` constraint declaration](#primary-key-constraint-declaration)
7. [`FOREIGN KEY` constraint declaration](#foreign-key-constraint-declaration)
8. [Indexes declaration](#indexes-declaration)

### `NOT NULL` constraint declaration
To declare `NOT NULL` constraint just annotate corresponding model field with `@NOT_NULL` annotation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 0)
@PRIMARY_KEY
@NOT_NULL // NOT NULL constraint declaration
public Long id;
{{< /highlight >}}
[Back to table of contents ^](#table-of-contents)

### `DEFAULT` constraint declaration
To declare `DEFAULT` constraint just annotate corresponding model field with `@DEFAULT` annotation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 3)
@DEFAULT(signedInteger = 28) // You can choose for options for default declaration, if nothing set than 0 value would be used
@NOT_NULL
public Integer defaultNumber;
{{< /highlight >}}
Without setting any fields of `@DEFAULT` annotation then default value for annotated field would be 0 (int, zero). KittyORM provide you some options on declaring default constraints, for example, you can set as a default value for field predefined `LiteralValues` enum element, signed integer, literal value or expression. Example of declaring `DEFAULT` constraint from predefined literals:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 4)
@DEFAULT(
        predefinedLiteralValue = LiteralValues.CURRENT_DATE
)
@NOT_NULL
public String creationDate;
{{< /highlight >}}
By design, KittyORM tries to insert NULL value from entity field to database at insert, so to avoid inserting NULL to database column that should acquire value from `DEFAULT` constraint before inserting model you have to invoke on this model `KittyModel.setFieldExclusion("modelFieldName")` method. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
IndexesAndConstraintsModel model = new IndexesAndConstraintsModel();
model.animal = someAnimal;
...
model.setFieldExclusion("creationDate"); // Forces KittyORM to exclude this field at insertion so DEFAULT constraint would be triggered
...
KittyMapper mapper = getDatabase().getMapper(IndexesAndConstraintsModel.class);
mapper.save(model);
mapper.close();
{{< /highlight >}}
[Back to table of contents ^](#table-of-contents)

### `UNIQUE` constraint declaration
You can declare `UNIQUE` constraint in two ways:

1. To declare `UNIQUE` constraint only on one column than just annotate model corresponding field with `@UNIQUE` annotation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 1)
@NOT_NULL
@UNIQUE
public Long rndId;
{{< /highlight >}}
In order to set conflict clause just set `@UNIQUE.onConflict` field with any suitable value from `ConflictClauses` enum.

2. To declare `UNIQUE` constraint on more than one column annotate model with `@UNIQUE_T` annotation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE(tableName = "cai")
@FOREIGN_KEY_T(
        name = "CAI_FK",
        columns = {IndexesAndConstraintsModel.RANDOM_ID_CNAME},
        reference = @FOREIGN_KEY_REFERENCE(
                foreignTableName = "random",
                foreignTableColumns = {"id"},
                onUpdate = OnUpdateDeleteActions.CASCADE,
                onDelete = OnUpdateDeleteActions.CASCADE
        )
)
@INDEX(indexColumns = {"creation_date"}) 
@UNIQUE_T(columns = {"rnd_id, animal"}) // Declaring unique constraint on more than two columns
public class IndexesAndConstraintsModel extends KittyModel {
    ...
}
{{< /highlight >}}
In order to set conflict clause just set `@UNIQUE_T.onConflict` field with any suitable value from `ConflictClauses` enum.  
In order to set `UNIQUE` constraint name set `@UNIQUE_T.name` otherwise it would be generated automatically.  
If you need more than one `UNIQUE` constraint declaration defined with usage of `@UNIQUE_T` annotation, annotate model with `UNIQUE_T_ARRAY`.

[Back to table of contents ^](#table-of-contents)

### `CHECK` constraint declaration
To declare `CHECK` constraint just annotate corresponding model field with `@CHECK` annotation and specify check expression. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 2)
@CHECK(checkExpression = "animal IN (\"CAT\", \"TIGER\", \"LION\")") // only cats allowed to this party
public Animals animal;
{{< /highlight >}}

[Back to table of contents ^](#table-of-contents)

### `COLLATE` constraint declaration
To declare `COLLATE` constraint just annotate corresponding model field with `@COLLATE` annotation and specify built-in collation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 2)
@COLLATE(collation = BuiltInCollations.NOCASE) // Collation example
@CHECK(checkExpression = "animal IN (\"CAT\", \"TIGER\", \"LION\")") 
public Animals animal;
{{< /highlight >}}

[Back to table of contents ^](#table-of-contents)

### `PRIMARY KEY` constraint declaration
You can declare `PRIMARY KEY` constraint in three ways:

1. To define `INTEGER PRIMARY KEY` just set `@KITTY_COLUMN.isIPK` field value to `true` of corresponding model field. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(
        columnOrder = 0, 
        isIPK = true
)
public Long id;
{{< /highlight >}}

2. Second way to define `PRIMARY KEY` is to annotate corresponding model field with `@PRIMARY_KEY` annotation. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 0)
@PRIMARY_KEY
@NOT_NULL
public Long id;
{{< /highlight >}}
In order to set order just set `PRIMARY_KEY.orderAscDesc` field with any suitable value from `AscDesc` enum.  
In order to set `AUTOIMCREMENT` flag set `@PRIMARY_KEY.autoincrement` field to true (default false).  
In order to set conflict clause just set `@PRIMARY_KEY.onConflict` field with any suitable value from `ConflictClauses` enum.  

3. Third way to define `PRIMARY KEY` is to annotate model implementation with `@PRIMARY_KEY_T` annotation and specify `@PRIMARY_KEY_T.columns` array. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE(tableName = "cpk_test")
@PRIMARY_KEY_T(
    columns = {"user_name", "email"}
)
public class CPKModel extends KittyModel {

    @KITTY_COLUMN(columnOrder = 0)
    public String userName;

    @KITTY_COLUMN(columnOrder = 1)
    @UNIQUE
    public String email;
    
    ...
}
{{< /highlight >}}
In order to set `PRIMARY KEY` constraint name set `@PRIMARY_KEY_T.name` otherwise it would be generated automatically.  
In order to set conflict clause just set `@PRIMARY_KEY_T.onConflict` field with any suitable value from `ConflictClauses` enum.

[Back to table of contents ^](#table-of-contents)

### `FOREIGN KEY` constraint declaration
You can declare `FOREIGN KEY` constraint in two ways:

1. If you have only one column at table that refer to another table column you can just annotate corresponding model field with `@FOREIGN_KEY` annotation and specify `@FOREIGN_KEY.reference` with `@FOREIGN_KEY_REFERENCE`. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_COLUMN(columnOrder = 1)
@NOT_NULL
@UNIQUE
@FOREIGN_KEY(
        reference = @FOREIGN_KEY_REFERENCE(
                foreignTableName = "random",
                foreignTableColumns = {"id"},
                onUpdate = OnUpdateDeleteActions.CASCADE,
                onDelete = OnUpdateDeleteActions.CASCADE
        )
)
public Long rndId;
{{< /highlight >}}

2. If you have more than one reference column at `FOREIGN KEY` declaration, annotate model with `@FOREIGN_KEY_T` annotation and specify `@FOREIGN_KEY_T.reference` with `@FOREIGN_KEY_REFERENCE` and `@FOREIGN_KEY_T.columns` with string array of reference columns. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE(tableName = "cai")
@FOREIGN_KEY_T(
        name = "CAI_FK",
        columns = {IndexesAndConstraintsModel.RANDOM_ID_CNAME},
        reference = @FOREIGN_KEY_REFERENCE(
                foreignTableName = "random",
                foreignTableColumns = {"id"},
                onUpdate = OnUpdateDeleteActions.CASCADE,
                onDelete = OnUpdateDeleteActions.CASCADE
        )
)
@INDEX(indexColumns = {"creation_date"})
public class IndexesAndConstraintsModel extends KittyModel {
    ...
    
    @KITTY_COLUMN(columnOrder = 1)
    @NOT_NULL
    @UNIQUE
    public Long rndId;

    ...
}
{{< /highlight >}}
In order to set `FOREIGN KEY` constraint name set `@FOREIGN_KEY_T.name` otherwise it would be generated automatically.  

If you need more than one **FK** that can be declared only with `@FOREIGN_KEY_T` you can annotate model implementation `@FOREIGN_KEY_T_ARRAY` and specify at `@FOREIGN_KEY_T_ARRAY.foreignKeys` all foreign keys you need.  
At `@FOREIGN_KEY_REFERENCE` annotation you have to specify reference table and columns by setting `@FOREIGN_KEY_REFERENCE.foreignTableName` and `@FOREIGN_KEY_REFERENCE.foreignTableColumns`. Optionally you can specify `ON UPDATE` and `ON DELETE` actions by setting `@FOREIGN_KEY_REFERENCE.onUpdate` and `@FOREIGN_KEY_REFERENCE.onDelete` with enum element from `OnUpdateDeleteActions`. Also, you can specify defferable option by setting `@FOREIGN_KEY_REFERENCE.deferrableOption` with some value `DeferrableOptions` enumeration.

{{% panel theme="warning" footer="Tip #1" %}}
Do not forget to turn on foreign keys supports by setting `@KITTY_DATABASE.isPragmaOn` to true at your KittyORM database implementation if you want to use them!
{{% /panel %}}

[Back to table of contents ^](#table-of-contents)

### Indexes declaration
In KittyORM indexes declarations stored at same POJO classes that are used for schema generation. To declare an index just annotate model implementation with columns that would be indexed with `@INDEX` annotation and set `@INDEX.indexColumns` with array of those indexed columns or in case when there is only one indexed column for one index declaration just annotate corresponding nodel implementation field with `@ONE_COLUMN_INDEX`. Example:

1. Index declaration with `@INDEX` annotation:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE(tableName = "cai")
@FOREIGN_KEY_T(
        name = "CAI_FK",
        columns = {IndexesAndConstraintsModel.RANDOM_ID_CNAME},
        reference = @FOREIGN_KEY_REFERENCE(
                foreignTableName = "random",
                foreignTableColumns = {"id"},
                onUpdate = OnUpdateDeleteActions.CASCADE,
                onDelete = OnUpdateDeleteActions.CASCADE
        )
)
@INDEX(indexColumns = {"creation_date"}) // index declaration
public class IndexesAndConstraintsModel extends KittyModel {
    ...

    @KITTY_COLUMN(columnOrder = 4)
    @DEFAULT(
            predefinedLiteralValue = LiteralValues.CURRENT_DATE
    )
    @NOT_NULL
    public String creationDate; // indexed column

    ...
}
{{< /highlight >}}

2. Index declaration with `@ONE_COLUMN_INDEX` annotation:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE(tableName = "cai")
...
public class IndexesAndConstraintsModel extends KittyModel {
    ...

    @KITTY_COLUMN(columnOrder = 5)
    @DEFAULT(
            predefinedLiteralValue = LiteralValues.CURRENT_TIMESTAMP
    )
    // One column indexe declaration example
    @ONE_COLUMN_INDEX(unique = true, indexName = "IAC_unique_index_creation_timestamp") 
    @NOT_NULL
    public Timestamp creationTmstmp;

    ...
}
{{< /highlight >}}

For both `@INDEX` and `@ONE_COLUMN_INDEX` index declaration you can specify index uniqueness (`@INDEX.unique` and `@ONE_COLUMN_INDEX.unique` fields), `IF NOT EXISTS` flag (`@INDEX.ifNotExists` and `@ONE_COLUMN_INDEX.ifNotExists` fields), where expression (`@INDEX.whereExpression` and `@ONE_COLUMN_INDEX.whereExpression` fields) and index name (`@INDEX.indexName` and `@ONE_COLUMN_INDEX.indexName` fields).  
If you need more than one index declaration with more than one indexed columns than annotate model implementation with `@INDEX_ARRAY` annotation and define your indexes there.

[Back to table of contents ^](#table-of-contents)
