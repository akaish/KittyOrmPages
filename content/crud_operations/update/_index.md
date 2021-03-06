---
title: "Updating existing entity"
date: 2018-12-15
draft: false
weight: 2
layout: "lesson"
---
For updating an entity with KittyORM you may use `KittyMapper.save(M model)` or `KittyMapper.update(M model)` method. As mentioned in the earlier lesson using save method force KittyORM to define what operation should be done with the provided model: update or insert. KittyORM makes this decision based on a state of entity fields that can be used for the unambiguous definition of associated record in the database.  
When you use `KittyMapper.update(M model)` method KittyORM would generate a query that sets values of a target table row with values from the model using generated `UPDATE` statement with `WHERE` clause. In the common case, rowid would be used for such statement. If the model has no rowid set than `UPDATE` statement would be generated with `WHERE` clause using model's PK values.

{{% panel footer="Demo application screenshots for this article" %}}
<center>![alt text](src/1.png?height=150px&classes=border,inline)  ![alt text](src/2.png?height=150px&classes=border,inline) ![alt text](src/3.png?height=150px&classes=border,inline)</center>
{{% /panel %}}

So, updating models with KittyORM is same as inserting them. Take a look at following code snippet:
{{< highlight java "linenos=inline, linenostart=1">}}
// Initializing database instance
BasicDatabase db = new BasicDatabase(getContext());
// Getting mapper instance
RandomMapper mapper = (RandomMapper) db.getMapper(RandomModel.class);
// Getting existing model from database (assuming that 0l model exists)
RandomModel toUpdate = mapper.findByIPK(0l);
// Setting model fields
toUpdate.randomInt = 12;
...
// Saving model with save method
mapper.save(toUpdate);
// Saving model with direct insert call
mapper.update(toInsert);
{{< /highlight >}}

However, what if you want to update more than one model? It is quite simple, instead of updating models in a bulk mode you can update some rows in a table with a usage of custom update condition within values to be updated set as fields of POJO. 
For example, you want to update all rows in `random` table where id is between (inclusively) 10 and 20 and set a random_int column to value 50. For this perform following steps:

1. Create new condition with the usage of `SQLiteConditionBuilder.class`:
{{< highlight java "linenos=inline, linenostart=1">}}
SQLiteConditionBuilder builder = new SQLiteConditionBuilder();
builder.addColumn("id")
       .addSQLOperator(SQLiteOperator.GREATER_OR_EQUAL)
       .addValue(10)
       .addSQLOperator(SQLiteOperator.AND)
       .addColumn("id")
       .addSQLOperator(SQLiteOperator.LESS_OR_EQUAL)
       .addValue(20);
{{< /highlight >}}
2. Create new `RandomModel.class` instance and set `randomInt` to 50:
{{< highlight java "linenos=inline, linenostart=1">}}
RandomModel toUpdate = new RandomModel();
toUpdate.randomInt = 50;
{{< /highlight >}}
3. Run `KittyMapper.update(M model, SQLiteCondition condition, String[] names, int IEFlag)` providing previosly created model, condition, `String` array of field names (`randomInt`) to set and inclusion\exclusion flag:
{{< highlight java "linenos=inline, linenostart=1">}}
mapper.update(toUpdate, builder.build(), new String[]{"randomInt"}, CVUtils.INCLUDE_ONLY_SELECTED_FIELDS);
{{< /highlight >}}

<details> 
  <summary>{{< icon name="fa-code" size="large" >}}Click to view update example in a block:</summary>
{{< highlight java "linenos=inline, linenostart=1">}}
// Initializing database instance
BasicDatabase db = new BasicDatabase(getContext());
// Getting mapper instance
RandomMapper mapper = (RandomMapper) db.getMapper(RandomModel.class);
// Creating condition builder instance
SQLiteConditionBuilder builder = new SQLiteConditionBuilder();
builder.addColumn("id")
       .addSQLOperator(SQLiteOperator.GREATER_OR_EQUAL)
       .addValue(10)
       .addSQLOperator(SQLiteOperator.AND)
       .addColumn("id")
       .addSQLOperator(SQLiteOperator.LESS_OR_EQUAL)
       .addValue(20);
// Creating blank model and setting it fields
RandomModel toUpdate = new RandomModel();
toUpdate.randomInt = 50;
// Updating table with custom clause and values from model
mapper.update(toUpdate, builder.build(), new String[]{"randomInt"}, CVUtils.INCLUDE_ONLY_SELECTED_FIELDS);
{{< /highlight >}}
</details>

{{% panel theme="warning" footer="Tip #1" %}}
You can path to `KittyMapper.update(M model, SQLiteCondition condition, String[] names, int IEFlag)` as `names` parameter column fields or POJO field names. With `IEFlag` you can specify how to treat values you passed as `names` parameter. Following flags are supported: `INCLUDE_ONLY_SELECTED_FIELDS`, `INCLUDE_ALL_EXCEPT_SELECTED_FIELDS`, `INCLUDE_ONLY_SELECTED_COLUMN_NAMES`, `INCLUDE_ALL_EXCEPT_SELECTED_COLUMN_NAMES` and `IGNORE_INCLUSIONS_AND_EXCLUSIONS`.
{{% /panel %}}

You can get more documentation on building clauses with `SQLiteConditionBuilder.class` at [KittyORM project page](https://akaish.github.io/KittyORMPages/).
