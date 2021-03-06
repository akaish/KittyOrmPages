---
title: "KittyORM datatypes mapping"
date: 2018-12-15
draft: false
weight: 50
layout: "lesson"
---
{{% panel footer="Demo application screenshots for this article" %}}
<center>![alt text](src/1.png?height=150px&classes=border,inline)&nbsp;&nbsp;![alt text](src/2.png?height=150px&classes=border,inline)</center>
{{% /panel %}}

By design instead of mapping java types to SQLite supported SQL datatypes KittyORM maps them directly into SQLite affinities, KittyORM supports predefined mapping for primitive java types, primitive wrappers, byte array, enumerations and following types: `java.lang.String`, `java.util.Date`, `java.util.Currency`, `java.util.Calendar`, `java.sql.Timestamp`, `java.math.BigInteger`, `java.math.BigDecimal`, `java.io.File`, `android.net.Uri`.  

**`TEXT` datatype affinity mapping**

SQLite affinity | Java type | Notes
--- | --- | ---
`TEXT` | Enumurations | To SQLite: `Enum someValue.name()`; from SQLite: `fieldType.getMethod("valueOf", String.class).invoke(fieldType, cursor.getString(someIndex))`
`TEXT` | `java.lang.String` | -
`TEXT` | `java.math.BigDecimal` | To SQLite: `BigDecimal someValue.toString()`; from SQLite: `new BigDecimal(cursor.getString(someIndex))`
`TEXT` | `android.net.Uri` | To SQLite: `Uri someValue.toString()`; from SQLite: `Uri.parse(cursor.getString(someIndex))`
`TEXT` | `java.io.File` | To SQLite: `File someValue.getAbsolutePath()`; from SQLite: `new File(cursor.getString(someIndex))`
`TEXT` | `java.math.BigInteger` | To SQLite: `BigInteger someValue.toString()`; from SQLite: `new BigInteger(cursor.getString(someIndex))`
`TEXT` | `java.util.Currency` | To SQLite: `Currency someValue.getCurrencyCode()`; from SQLite: `Currency.getInstance(cursor.getString(someIndex))`

**`INTEGER` datatype affinity mapping**

SQLite affinity | Java type | Notes
--- | --- | ---
`INTEGER` | `java.util.Calendar` | To SQLite: `Calendar someValue.getTimeInMillis()`; from SQLite: `Calendar.getInstance().setTimeInMillis( cursor.getLong(someIndex) )`
`INTEGER` | `java.sql.Timestamp` | To SQLite: `Timestamp someValue.getTime()`; from SQLite: `new Timestamp(cursor.getLong(someIndex))`
`INTEGER` | `java.util.Date` | To SQLite: `Date someValue.getTime()`; from SQLite: `new Date(cursor.getLong(someIndex))`
`INTEGER` | `byte` | -
`INTEGER` | `int` | -
`INTEGER` | `long` | -
`INTEGER` | `short` | -
`INTEGER` | `boolean` | To SQLite: `boolean someValue ? 1 : 0`; from SQLite: `cursor.getInt(someIndex) == 1`
`INTEGER` | `java.lang.Byte` | -
`INTEGER` | `java.lang.Integer` | -
`INTEGER` | `java.lang.Long` | -
`INTEGER` | `java.lang.Short` | -
`INTEGER` | `java.lang.Boolean` | To SQLite: `Boolean someValue ? 1 : 0`; from SQLite: `cursor.getInt(someIndex) == 1`

**`REAL` datatype affinity mapping**

SQLite affinity | Java type | Notes
--- | --- | ---
`REAL` | `float` | -
`REAL` | `double` | -
`REAL` | `java.lang.Float` | -
`REAL` | `java.lang.Double` | -

**`NONE` datatype affinity mapping**

SQLite affinity | Java type | Notes
--- | --- | ---
`NONE` | `byte[]` | -
`NONE` | `java.lang.Byte[]` | -

### Custom mapping rules
By default, KittyORM provides mapping of most java types that you may want to use. However, KittyORM also offers some functionality for user defined mapping rules e.g. you can tell KittyORM how to store and retrieve any java objects you want. For those purposes KittyORM has `KITTY_COLUMN_SERIALIZATION` annotation with what you can achieve storing your objects or objects data as `TEXT` or `NONE` SQLite datatype affinities at your database. To use it, take following steps:

1. Define at `KittyModel.class` implementation model table field with SQLite datatype affinity specified explicitly (`TypeAffinities.TEXT`, `TypeAffinities.BLOB` or `TypeAffinities.NONE` only).
{{< highlight java "linenos=inline, linenostart=1">}}
// Saving to text
@KITTY_COLUMN(
        columnOrder = 18,
        columnAffinity = TypeAffinities.TEXT
)
@KITTY_COLUMN_SERIALIZATION
public AnimalSounds stringSDF;

// Saving to blob
@KITTY_COLUMN(
        columnOrder = 20,
        columnAffinity = TypeAffinities.BLOB
)
@KITTY_COLUMN_SERIALIZATION
public Bitmap byteArraySDF;
{{< /highlight >}} 

2. Write your methods how to transform your object to string\blob and back. If `serializationMethodName` or `deserializationMethodName` of `KITTY_COLUMN_SERIALIZATION` were not specified explicitly than KittyORM would try to call method `String\byte[] "fieldname" + Serialize` for serialization (no parameters) and `YourType "fieldName" + Deserialize` for deserialization (`String\byte[] fromCursor` as parameter). For example, for model field `AnimalSounds stringSDF` default serialization method would be `String stringSDFSerialize()` and default deserialization method would be `AnimalSounds stringSDFDeserialize(String cvData)`.
{{< highlight java "linenos=inline, linenostart=1">}}
String stringSDFSerialize() {
    if(stringSDF == null) return null;
    return new GsonBuilder().create().toJson(stringSDF);
}

AnimalSounds stringSDFDeserialize(String cvData) {
    if(cvData == null) return null;
    if(cvData.length() == 0) return null;
    return new GsonBuilder().create().fromJson(cvData, AnimalSounds.class);
}

public byte[] byteArraySDFSerialize() {//byteArraySDFSerialize
    if(byteArraySDF == null) return null;
    ByteArrayOutputStream bmpStream = new ByteArrayOutputStream();
    byteArraySDF.compress(Bitmap.CompressFormat.PNG, 100, bmpStream);
    return bmpStream.toByteArray();
}

public Bitmap byteArraySDFDeserialize(byte[] cursorData) {
    if(cursorData == null) return null;
    if(cursorData.length == 0) return null;
    return BitmapFactory.decodeByteArray(cursorData, 0, cursorData.length);
}
{{< /highlight >}} 

Now you are ready for using custom mapping rules with KittyORM. Congratulations.
