---
title: "Debugging KittyORM"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 140
---
By design KittyORM able to log to logstream quite a lot of information in order to make it easier to implement KittyORM database in your application. Right now KittyORM has three types of logs: base log that logs main information such as errors and bootstrap messages, query log that logs queries sent to be executed by SQLite and dex utility log that logs what happens when KittyORM registry created from package.  
Logging can be enabled via `KITTY_DATABASE` annotation. To do this, set up return values of `isLoggingOn`, `isProductionOn`, `isKittyDexUtilLoggingEnabled` and `logTag` of `KITTY_DATABASE` annotation that annotates your KittyORM implementation database class.
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_DATABASE(
        isLoggingOn = true, // Base logging flag
        isProductionOn = false, // Production logging flag
        isKittyDexUtilLoggingEnabled = false, // dex logging flag
        logTag = MigrationDBv3.LTAG, // log tag
        databaseName = "mig", // database name
        databaseVersion = 3, // database version
        ...
)

public class MigrationDBv3 extends KittyDatabase {

    public static final String LTAG = "MIGv3";
    
    ...
}
{{< /highlight >}}
Let's take a closer look on base logging setting:

* `isLoggingOn` - main logging flag, by default - false. Setting this value to true would cause KittyORM log all errors, warnings and bootstrap info except log output of dex util and queries. When this value is false than no logging would be performed at all, even if `isProductionOn` flag is false and `isKittyDexUtilLoggingEnabled` flag is true. Most log messages would contain specified log tag, database name and database version to make debugging easier.
* `isProductionOn`- query logging, by default - true. When this value is false and `isLoggingOn` is true - logs queries sent to be executed by SQLite to log stream. Be sure that you turn production mode on when you're ready to publish your application because logging queries to log stream is a potential security vulnerability and may slow down KittyORM when, for example, you need to save a big amount of entities to database.
* `isKittyDexUtilLoggingEnabled` - dex util logging flag, by default - false. When this value is true and `isLoggingOn` is true - logs messages related with usage of `KittyDexUtils.class`. Actually, it is bag idea to generate KittyORM registry from package at production builds, because it causes slow initialization of KittyORM implementation, refer to [Speed up your KittyORM article](https://akaish.github.io/KittyORMPages/speedup/) for more details.
* `logTag` - log tag for all log messages related to this KittyORM database implementation. By default - `"KittySQLiteORM"`.

Also, you may make your query log more informative by overloading `String toLogString()` method of `KittyModel.class` implementation to log update and insert queries. Example:
{{< highlight java "linenos=inline, linenostart=1">}}
@KITTY_TABLE
public class SimpleExampleModel extends KittyModel {
    public SimpleExampleModel() {
        super();
    }

    @KITTY_COLUMN(
            isIPK = true,
            columnOrder = 0
    )
    public Long id;

    @KITTY_COLUMN(columnOrder = 1)
    public int randomInteger;

    @KITTY_COLUMN(columnOrder = 2)
    public String firstName;

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder(64);
        return sb.append("[ rowid = ")
                    .append(getRowID())
                    .append(" ; id = ")
                    .append(id)
                    .append(" ; randomInteger = ")
                    .append(randomInteger)
                    .append(" ; firstName = ")
                    .append(firstName)
                    .append(" ]")
                    .toString();
    }

    public String toLogString() {
        return toString();
    }
}
{{< /highlight >}}
