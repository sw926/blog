---
title: Android Room
date: 2018-06-13 17:16:23
category: Android
tags: 
    - Android
    - Room
---

## 说明

Android Room 作为 Android Architecture 的 orm 部分，接入是非常简单的。

首先明确 Room 能做什么，简单概括，能让你把一行 SQL 语句变成对象，是现在最好用的 ORM 框架，当然这是废话，官方能拿的出来，肯定要比第三方的要好。首先体验一下：

简单的插入数据，不用写 SQL:
``` kotlin
@Insert(onConflict = OnConflictStrategy.REPLACE)
fun insertAll(vararg record: Record)

@Insert(onConflict = OnConflictStrategy.REPLACE)
fun insert(record: Record)
```
查询数据，一行 SQL，一个函数声明搞定

``` kotlin
@Query("SELECT * FROM Record")
fun getAll(): List<Record>
```

而且 SQL 是有**代码提示**和**语法检查**的。

下面，我们在现有项目上使用 Room，使用新的orm框架，而不用对数据库结构做任何修改。

## 添加 Room

``` gradle
def room_version = "1.1.0" // or, for latest rc, use "1.1.1-rc1"

implementation "android.arch.persistence.room:runtime:$room_version"
kapt "android.arch.persistence.room:compiler:$room_version"
```

## Entity

现在的项目数据量有个 Record table，创建的 SQL 语句为

``` sql
CREATE TABLE Record (
    _id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    date INTEGER UNIQUE NOT NULL,
    records TEXT,
    need_sync INTEGER DEFAULT 0
);
```
主键为 `_id`，`data` 为整型，存储的是时间，`records` 为字符串，是一个 json 对象，`need_sync` 是一个 Boolean，以整型存储在数据库中。

我想做的：
 - `_id` 在数据类中要名字是 `id`
 - `date` 为 UNIQUE
 - `date` 直接读取为 LocalDate 对象
 - `records` 直接读取为 Bean 对象
 - `need_sync` 直接读取为 Boolean，数据类中的名字当然也要改

最终的 Entity 类声明为：

``` kotlin
@Entity(tableName = "Record", indices = [(Index(value = arrayOf("date"), unique = true))])
class Record {
    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "_id")
    var id: Long = 0

    var date: LocalDate? = null
    var records: Bean? = null

    @ColumnInfo(name = "need_sync")
    var needSync = false
}
```

除了类声明，我们不需要 SQL 语句去创建 Table

## Dao

RecordDao，用来访问数据对象：

``` kotlin
@Dao
interface RecordDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertAll(vararg record: Record)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insert(record: Record)

    @Delete
    fun delete(record: Record)

    @Delete
    fun deleteAll(vararg record: Record)

    @Update
    fun update(record: Record)

    @Query("SELECT * FROM Record")
    fun getAll(): List<Record>

    @Query("SELECT * FROM Record WHERE date = :date")
    fun getByDate(date: LocalDate): List<Record>
}
```
直接的插入、删除、更新是不用写 SQL，自定义的查询还是需要写 SQL，可以直接在 SQL 语句中绑定函数参数，只要在参数名字前加 “:” 就可以了，像 `getByDate`，`:date` 就是标识函数的 date 参数就是 SQL 中的参数：

``` kotlin
@Query("SELECT * FROM Record WHERE date = :date")
fun getByDate(date: LocalDate): List<Record>
```

`@Query` 里面的 SQL 语句不仅有代码提示，而且有语法检查，写错 table name 和 函数参数名都会报错的，手残党的福音啊。

## Database

Entity 和 Dao 都声明好了，现在要创建数据库了，还记得之前怎么做的吗：

 - 继承 SQLiteOpenHelper
 - `onCreate` 执行 SQL 语句创建数据库
 - `onUpgrade` 进行 Migration
 - `execSQL` 获取 `Cursor`
 - 把 `Cursor` 转换为对象

如果使用 Room 呢？

没有 SQLiteOpenHelper 了，也没有 `onCreate` 了：

``` kotlin
@Database(entities = [(Record::class)], version = 10)
abstract class AppDataBase : RoomDatabase() {

    abstract fun recordDao(): RecordDao
}
```

数据库创建的声明完成了，使用了 Room，要接受这几点：

 - 不需要 `SQLiteOpenHelper`
 - 不需要 `Cursor`
 - 可能连 `SQLiteDatabase` 也不需要

## TypeConverters

上面是创建数据库的全部代码了吗？当然不是，我们还需要 TypeConverters 和 Migration，

date 在数据里面以 Long 的形式存储，读取的时候需要转换为 LocalDate 对象，存储的时候需要把 LocalDate 转换为 Long，对于一种转换关系，我们只需要声明一个静态函数就可以了，名字随意，参数和返回值的类型为对应的转换关系。

``` kotlin
class DbTypeConverters {
    companion object {

        @TypeConverter
        @JvmStatic
        fun toLocalDate(value: Long): LocalDate =
                LocalDateTime.ofInstant(Instant.ofEpochMilli(value), ZoneId.systemDefault()).toLocalDate()

        @TypeConverter
        @JvmStatic
        fun toLocalDate(value: LocalDate): Long =
                value.atStartOfDay(ZoneId.of("UTC")).toInstant().toEpochMilli()
    }

}
```

然后把 TypeConverters 添加到 Database

``` kotlin
@Database(entities = [(Record::class)], version = 10)
@TypeConverters(DbTypeConverters::class)
abstract class AppDataBase : RoomDatabase() {

    abstract fun recordDao(): RecordDao
}
```

## Migrations

数据库升级，我们需要声明的是一个 Migration，

``` kotlin
object PeriodDbMigration {

    @JvmField
    val MIGRATION_1_2 = object : Migration(1, 2) {

        override fun migrate(database: SupportSQLiteDatabase) {
            // update
        }

    }

    @JvmField
    val MIGRATION_3_4 = object : Migration(3, 4) {

        override fun migrate(db: SupportSQLiteDatabase) {
            // update
        }

    }
}
```

和 SQLiteOpenHelper 的 onUpgrade 一样，只不过每次升级拆分为一次 Migration 操作。

然后就是创建数据库：

通过 addMigrations 添加数据库升级的操作，

``` kotlin
val database = Room.databaseBuilder(application, PeriodDataBase.class, "database.db")
                .addMigrations(PeriodDbMigration.MIGRATION_1_2, PeriodDbMigration.MIGRATION_3_4)
                .allowMainThreadQueries()
                .build();
```

`allowMainThreadQueries()` 是允许在主线程上进行操作，对于之前在主线程上读写数据的同学们，先加上 `allowMainThreadQueries()`，然后慢慢优化吧。

## 使用

一般会把数据作为单例使用，然后调用 Dao 中函数就可以了：

``` kotlin
val all = database.recordDao().getAll()
```