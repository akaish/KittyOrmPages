---
title: "Finding entities"
date: 2018-12-15
draft: false
weight: 4
layout: "lesson"
---
{{% panel footer="Demo application screenshots for this article" %}}
<center>![alt text](src/1.png?height=150px&classes=border,inline)  ![alt text](src/2.png?height=150px&classes=border,inline) ![alt text](src/3.png?height=150px&classes=border,inline) ![alt text](src/4.png?height=150px&classes=border,inline)</center>
{{% /panel %}}

To find entities with KittyORM just use one of methods already implemented at [`KittyMapper.class`](https://akaish.github.io/KittyORM/net/akaish/kitty/orm/KittyMapper.html):
{{< highlight java "linenos=inline, linenostart=1">}}
// Initializing database instance
BasicDatabase db = new BasicDatabase(getContext());
// Getting mapper instance
RandomMapper mapper = (RandomMapper) db.getMapper(RandomModel.class);
// Getting existing model from database (assuming that 0l model exists)
RandomModel byIPK = mapper.findByIPK(0l);
// Getting existing model with rowid (assuming that 10l model exists)
RandomModel byRowid = mapper.findByRowID(10l);
// Getting all models
List<RandomModel> all = mapper.findAll();
// Getting model with condition (fetching 100 existing tigers)
QueryParameters parameters = new QueryParameters();
parameters.setOffset(0l).setLimit(100l);
SQLiteConditionBuilder builder = new SQLiteConditionBuilder();
builder.addField(AbstractRandomModel.RND_ANIMAL_CNAME)
       .addSQLOperator(SQLiteOperator.EQUAL)
       .addValue(Animals.TIGER.name());
List<RandomModel> hundredOfTigers = mapper.findWhere(builder.build(), parameters);
// Getting model with condition (fetching 100 existing tigers with SQLite string condition)
hundredOfTigers = mapper.findWhere(parameters, "#?randomAnimal = ?", Animals.TIGER.name());
{{< /highlight >}}

**Already implemented find methods in KittyORM:**

Method name | Method description
--- | --- 
`findWhere(SQLiteCondition where, QueryParameters qParams)` | Returns list of models associated with records in backed database table that suits provided clause and query parameters.
`findWhere(QueryParameters qParams, String where, Object... conditionValues)` | Returns list of models associated with records in backed database table that suits provided clause and query parameters.
`findWhere(SQLiteCondition where)` | Returns list of models associated with records in backed database table that suits provided clause.
`findWhere(String where, Object... conditionValues)` | Returns list of models associated with records in backed database table that suits provided clause.
`findAll(QueryParameters qParams)`  | Returns list of all models associated with records in backed database table with usage of passed qParams.
`findAll()` | Returns list of all models associated with records in backed database table.
`findByRowID(Long rowid)` | Returns model filled with data from database or null if no record with provided rowid found.
`findByPK(KittyPrimaryKey primaryKey)` | Returns model filled with data from database or null if no record with provided PK found.
`findByIPK(Long ipk)` | Returns model filled with data from database or null if no record with provided IPK found.
`findFirst(SQLiteCondition where)` | Returns first record in KittyModel wrapper in database table that suits provided condition.
`findFirst()` | Returns first record in KittyModel wrapper in database table.
`findLast(SQLiteCondition where)` | Returns last record in KittyModel wrapper in database table that suits provided condition.
`findLast()` | Returns last record in KittyModel wrapper in database table.

{{% panel theme="warning" footer="Tip #1" %}}
Do not count collection received from `findAll()` method if you want to count all records in table, use already implemented **sum** and **count** methods!
{{% /panel %}}

See following count and sum code example snippet:
{{< highlight java "linenos=inline, linenostart=1">}}
// Initializing database instance
BasicDatabase db = new BasicDatabase(getContext());
// Getting mapper instance
RandomMapper mapper = (RandomMapper) db.getMapper(RandomModel.class);
// Count all records in database
long count = mapper.countAll();
// Count all dogs
SQLiteConditionBuilder builder = new SQLiteConditionBuilder();
builder.addField(AbstractRandomModel.RND_ANIMAL_CNAME)
       .addSQLOperator(SQLiteOperator.EQUAL)
       .addValue(Animals.DOG.name());
long dogsCount = mapper.countWhere(builder.build());
// Sum all dog's random_int
long dogsRndIntSum = mapper.sum("random_int", "#?randomAnimal = ?", Animals.DOG.name());
{{< /highlight >}}

### Implementing extended CRUD controller
OK, this is simple, already implemented by `KittyMapper` methods allows you to do most of the database related operations, however you may want to extend `KittyMapper` CRUD controller and implement some methods that would be used commonly in your application. Here some steps to take:

1. Define in KittyORM registry CRUD controller to be used with particular model by defining it in your `KittyDatabase` implementation class with usage `@KITTY_DATABASE_REGISTRY` at `domainPairs` or by defining it at your model implementation class with usage of `@KITTY_EXTENDED_CRUD` annotation:
{{< highlight java "linenos=inline, linenostart=1">}}
// Defining at registry example
@KITTY_DATABASE(
        databaseName = "basic_database",
        domainPackageNames = {"net.akaish.kittyormdemo.sqlite.basicdb"},
        ...
)
@KITTY_DATABASE_REGISTRY(
        domainPairs = {
                @KITTY_REGISTRY_PAIR(model = ComplexRandomModel.class, mapper = ComplexRandomMapper.class),
                @KITTY_REGISTRY_PAIR(model = IndexesAndConstraintsModel.class),
                @KITTY_REGISTRY_PAIR(model = RandomModel.class, mapper = RandomMapper.class) // registry CRUD controller definition
        }
)
public class BasicDatabase extends KittyDatabase {
    ...
}
// Defining at model example
@KITTY_TABLE
@KITTY_EXTENDED_CRUD(extendedCrudController = RandomMapper.class) // model CRUD controller definition
@INDEX(
        indexName = "random_animal_index",
        indexColumns = {AbstractRandomModel.RND_ANIMAL_CNAME}
)
public class RandomModel extends AbstractRandomModel {
    ...
}
{{< /highlight >}}

2. Create new class that extends `KittyMapper.class`, implement default constructor and locate it at domain package, fill it with your logic.
<details> 
  <summary>{{< icon name="fa-code" size="large" >}}Click to view extended crud controller implementation example: </summary>
{{< highlight java "linenos=inline, linenostart=1">}}
public class RandomMapper extends KittyMapper {

    public <M extends KittyModel> RandomMapper(KittyTableConfiguration tableConfiguration,
                                              M blankModelInstance,
                                              String databasePassword) {
        super(tableConfiguration, blankModelInstance, databasePassword);
    }

    protected SQLiteCondition getAnimalCondition(Animals animal) {
        return new SQLiteConditionBuilder()
                .addColumn(RND_ANIMAL_CNAME)
                .addSQLOperator("=")
                .addObjectValue(animal)
                .build();
    }

    public long deleteByRandomIntegerRange(int start, int end) {
        return deleteWhere("#?randomInt >= ? AND #?randomInt <= ?", start, end);
    }

    public long deleteByAnimal(Animals animal) {
        return deleteWhere(getAnimalCondition(animal));
    }

    public List<RandomModel> findByAnimal(Animals animal, long offset, long limit, boolean groupingOn) {
        SQLiteCondition condition = getAnimalCondition(animal);
        QueryParameters qparam = new QueryParameters();
        qparam.setLimit(limit).setOffset(offset);
        if(groupingOn)
            qparam.setGroupByColumns(RND_ANIMAL_CNAME);
        else
            qparam.setGroupByColumns(KittyConstants.ROWID);
        return findWhere(condition, qparam);
    }

    public List<RandomModel> findByIdRange(long fromId, long toId, boolean inclusive, Long offset, Long limit) {
        SQLiteCondition condition = new SQLiteConditionBuilder()
                .addColumn("id")
                .addSQLOperator(inclusive ? GREATER_OR_EQUAL : GREATER_THAN)
                .addValue(fromId)
                .addSQLOperator(AND)
                .addColumn("id")
                .addSQLOperator(inclusive ? LESS_OR_EQUAL : LESS_THAN)
                .addValue(toId)
                .build();
        QueryParameters qparam = new QueryParameters();
        qparam.setLimit(limit).setOffset(offset).setGroupByColumns(KittyConstants.ROWID);
        return findWhere(condition, qparam);
    }

    public List<RandomModel> findAllRandomModels(Long offset, Long limit) {
        QueryParameters qparam = new QueryParameters();
        qparam.setLimit(limit).setOffset(offset).setGroupByColumns(KittyConstants.ROWID);
        return findAll(qparam);
    }

}
{{< /highlight >}} 

</details>

That's all, after this when you call `getMapper(Class modelClass)` of your `KittyDatabase` implementation you would receive ready to go your extended CRUD controller.
