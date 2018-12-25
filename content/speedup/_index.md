---
title: "Speed up your KittyORM"
date: 2018-12-15
draft: false
layout: "lesson"
weight: 120
---
KittyORM build with Java7 for Android and main idea is to implement developer-friendly tool that would save your time and money on implementing business logic that needs SQLite database for storing data. KittyORM provides a lot of features that can save your time on developing and supporting your data model, but it uses reflection to avoid messing around with raw SQL code and some tools as generating data model from domain package. And it is not performance free. This tutorial page is kind of FAQ or cookbook of how to tune up your KittyORM implementation to achieve maximum performance. Main idea of this tutorial - use only those features you really need. And now goes list of tips.
### Tip №1: Avoid using generating data model from packages
KittyORM supports generating data model from classes that implements `KittyModel.class` and `KittyMapper.class`. It's really handy feature, but slow. While generating list of classes in application namespace `KittyDexUtils.class` has to scan all classes that exist, check naming rules, get their instances with `Class.forName(className, false, context.getClassLoader());` and check if classes are assignable from base model or mapper class, check if their package is domain package, check if those classes suits `KittyDexClassFilter.class` instance. It is very performance expensive operation, because typically there are hundreds or even thousands of classes that are available in application namespace.  
Use this feature only for development or testing purposes and on production it is better to define your data model with usage of `KITTY_DATABASE_REGISTRY` and `KITTY_REGISTRY_PAIR` annotations or by initializing static KittyORM registry collection and returning it in `KittyDatabase` implementation via overloaded `KittyDatabase.getStaticRegistryRegistry()` method.
### Tip №2: Avoid multiply initialization of `KittyDatabase`
Each new instance of `KittyDatabase` would execute a lot of operations on generating data model, registry, create and drop schema SQLite code etc. So, better approach would be place one initialized on demand instance of your KittyORM database into [Application class](https://developer.android.com/reference/android/app/Application) and work only with this one instance.
### Tip №3: Optimize your data model and statements
That's simple. You have a lot of `SELECT` queries on table with condition on some field? Index it. Build more efficient conditions for your statements. For example, you can follow [this article](https://www.sqlite.org/optoverview.html) for instructions related with optimizing your queries.  
You want to insert a lot of entities? Use insert in transaction feature for `DELETE`, `UPDATE` and `INSERT` statements. For example, for insertions use `KittyMapper.insertInTransaction(List<M> models)`. This would cause execution your queries at one time instead of forcing SQLite run each of your statement separately, so in some cases using this tip would cause up to 20x faster execution of insertions.
### Tip №4: Do not use `WITHOUT ROWID` flag for your tables
KittyORM uses `rowid` field for indication if this model is new or existing. If model corresponds to table that was created with `WITHOUT ROWID` flag than KittyORM would have to run much more operations related with fetching synthetic or natural primary key value(s) in order to differ models.
### Tip №5: Use predefined drop, create and migration scripts at production
While developing your data model step by step KittyORM would provide you good tool for generating scripts for creating schema, dropping schema and upgrading schema from version to version with usage of `KittySimpleMigrationScriptGenerator Migrator`. However, you can slightly decrease initialization times of your `KittyDatabase` implementation at production by using static scripts initialized in your `KittyDatabase` implementation instance or by using those scripts stored at assets or file system. Just develop your data model, save generated by KittyORM schema scripts and use them instead of generating those scripts each time when you initialize new instance of your `KittyDatabase` implementation.
### Tip №6: Turn off logging at production
OK, you're ready for production. Do not forget to turn off logging by setting `isLoggingOn()`, `isProductionOn()` and `isKittyDexUtilLoggingEnabled()` flags of `KITTY_DATABASE` annotation to false, true and false respectively. You may leave `isLoggingOn()` flag on, however it is good idea to turn on `isProductionOn()` flag because all of your queries would be logged to log stream that slows execution of statements and is a potential security vulnerability.
### Tip №7: Run expensive operations not in UI thread
And last tip: run your database related operations in another thread, especially if there is chance that they can be expensive. Good practice is to run all database related operations in [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask) or use any other option to avoid execution of database code in UI thread. Why? Because SQLite is not as fast as light and KittyORM is another layer of code that needs resources for execution too. So using [AsyncTask](https://developer.android.com/reference/android/os/AsyncTask) would be good idea to avoid UI lags and even ANR for long operations.