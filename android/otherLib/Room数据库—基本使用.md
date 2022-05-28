# Room数据库—基本使用

​		Room是持久型数据库，在SQLite的基础上提供了一个抽象层，Room解决简化数据库设置、配置以及与数据库交互方面的琐碎工作，让我们使用SQLite更简单。

​		SQLite3数据库在移动开发中不太常使用，是由于数据库维护需要很大的成本，为了快速迭代，数据库也是可以省略的存在，使用Key-Value键值对就可以满足我们开发中遇到90%以上的数据存储问题，Room是为了解决开发中的不友好操作，节约使用成本而诞生的。但也不可否认的是App需要更好的优化，数据库是必不可少的一项。上一篇解决了使用问题，这篇是为了解决使用中遇到的一些疑问，深入了解Room的使用。

会按照以下路线进行分析Room

* 基本使用
* 注解类解释
* LiveData、RxJava、协程的支持
* 数据库迁移



## 基本使用

Room的存在就是为了简化我们使用数据库的成本而存在的，所以使用入门很简单，使用可以分为以下3个模块

* 库
* 表
* 操作

接下来我们创建一个基本的数据库单元：`表`

```kotlin
package com.hilary.database.entity

import androidx.room.Entity
import androidx.room.PrimaryKey

const val LOCATION_TABLE = "location"
//数据库表名
@Entity(tableName = LOCATION_TABLE)
data class LocationEntity(
    //自增ID
    @PrimaryKey(autoGenerate = true)
    val id: Int,
    //经度
    var longitude: Long = 0,
    //纬度
    var latitude: Long = 0)
```

创建一个表对应的操作类：`Dao`

```kotlin
package com.hilary.database.dao

import androidx.lifecycle.LiveData
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import com.hilary.database.entity.LOCATION_TABLE
import com.hilary.database.entity.LocationEntity
import java.util.concurrent.Flow

@Dao
interface LocationDao {
    @Insert
    fun insert(location: LocationEntity)

    @Query("SELECT * FROM $LOCATION_TABLE WHERE id= :id")
    fun getId(id: Int): LocationEntity?

    @Query("SELECT * FROM $LOCATION_TABLE WHERE id= :id")
    fun getIdByLiveData(id: Int): LiveData<LocationEntity?>

}
```

创建数据库

```kotlin
package com.hilary.database

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import com.hilary.database.dao.LocationDao
import com.hilary.database.entity.LocationEntity

private const val DATABASE_NAME = "location_db"

@Database(entities = [LocationEntity::class], version = 1, exportSchema=false)
abstract class FMDataBaseRoom: RoomDatabase() {
		//定义操作对象变量，Room会帮我们生成同名的方法，只需要调用同名方法就可以获取是Dao对象
    abstract val locationDao: LocationDao

    companion object {
        @Volatile
        private var INSTANCE: FMDataBaseRoom? = null
        fun getInstance(context: Context): FMDataBaseRoom? {
            var instance = INSTANCE
            if (instance == null) {
                instance = Room.databaseBuilder(
                    context.applicationContext,
                    FMDataBaseRoom::class.java,
                    DATABASE_NAME,
                ).fallbackToDestructiveMigration()
                    .build()
                INSTANCE = instance
            }
            return INSTANCE
        }
    }
}
```

使用

```kotlin
val instance = FMDataBaseRoom.getInstance(this)
val location = LocationEntity(0, 1000, 11000)
Thread{
  instance?.locationDao?.insert(location)
}.start()
```

Room使用就这么简单，一般情况下，我们并不会使用一些复杂的操作，简单的操作就可以满足我们日常开发的需求，但想要深入了解，需要从Room的注解入手。Room几乎实现了SQLite3的全部功能，数据库具体都能帮我们做什么，可以去查看SQLite3文档：https://www.sqlite.org/index.html

## Room注解

`Room-common`是Room注解库

<img src="http://qike.hilary.top/uPic/image-20220505164801171.png" alt="image-20220505164801171" style="zoom: 50%;" />

下面我们看下注解类

### 表注解

* Entity

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.CLASS)
  public @interface Entity {
      /**
       * 数据库表名，如果没有设置使用类名作为表名
       *
       */
      String tableName() default "";
  
      /**
       * 数据库表的索引
       */
      Index[] indices() default {};
  
      /**
       * 是否继承索引，如果需要继承父类的索引，需要设置此值为true，子类默认是不会继承父类中的索引设置的
       */
      boolean inheritSuperIndices() default false;
  
      /**
       * 批量添加主键
       * 在字段上添加使用 @PrimaryKey
       */
      String[] primaryKeys() default {};
  
      /**
       * 批量添加外键
       * 在字段上添加使用 @ForeignKey
       */
      ForeignKey[] foreignKeys() default {};
  
      /**
       * 批量添加忽略字段
       * 在字段上添加使用 @Ignore
       */
      String[] ignoredColumns() default {};
  }
  ```

* Index

  ```java
  public @interface Index {
      /**
       * 索引集合
       */
      String[] value();
  
      /**
       * 索引排序集合，与索引数量一致，所有索引的默认排序为升序
       * 
       */
      Order[] orders() default {};
  
      /**
       * 索引名称，如果没有设置默认为：index_${tableName},如果是两条件索引，则向名称后面以`_`分隔追加索引字段名称
       */
      String name() default "";
  
      /**
       * 索引是否设置唯一值
       */
      boolean unique() default false;
  
      enum Order {
          /**
           * 升序
           */
          ASC,
  
          /**
           * 降序
           */
          DESC
      }
  }
  ```

* Primarykey

  ```java
  public @interface PrimaryKey {
      /**
       * 设置字段是否为自增
       * 字段类型必须为int或long
       */
      boolean autoGenerate() default false;
  }
  ```

* ForeignKey

  ```java
  public @interface ForeignKey {
      /**
       * 	外键类设置，必须和@Entity在同一个数据库中
       */
      Class<?> entity();
  
      /**
       * 外键父类列的集合
       * 必须与childColumns集合数量一致
       */
      String[] parentColumns();
  
      /**
       * 外键子类列的表集合
       * 必须与parentColumns集合数量一致
       */
      String[] childColumns();
  
      /**
       * 当父类数据从数据库中删除后，关联删除子类的数据，默认不做任何关联操作
       */
      @Action int onDelete() default NO_ACTION;
  
      /**
       * 当父类数据有变化，关联子类数据同时更新，默认不做任何关联操作
       */
      @Action int onUpdate() default NO_ACTION;
  
      /**
       * 本表的某条数据发生变动时，关联的数据是否立即执行变更，默认是立即执行
       */
      boolean deferred() default false;
  
      /**
       * 关联操作不做任何操作
       */
      int NO_ACTION = 1;
  
      /**
       * 在执行删除或更新操作时，如果有子类映射关系时，父类无法做相相应操作，
       * 只有没有子类映射时才可以正常做delete或update操作。
       */
      int RESTRICT = 2;
  
      /**
       * 如果父类中的数据做删除或更新操作涉及到子类中的记录时，则子类与父类关联的字段值更新为Null，如果字段设置不允许为null侧报错
       * 父类数据操作失败
       */
      int SET_NULL = 3;
  
      /**
       * 和SET_NULL类似，默认值用null代替
       */
      int SET_DEFAULT = 4;
  
      /**
       * 当父类数据删除时，同时会删除与子类关联的数据，数据更新时同时也会更新关联的子类数据
       */
      int CASCADE = 5;
  
      @IntDef({NO_ACTION, RESTRICT, SET_NULL, SET_DEFAULT, CASCADE})
      @Retention(RetentionPolicy.CLASS)
      @interface Action {
      }
  }
  ```

  我们可以根据不同的需求使用不同的操作事件。

* ColumnInfo

  自定义列信息类，可以通过此注解来指定列名，默认值，值类型

  ```java
  public @interface ColumnInfo {
      /**
       * 列名
       */
      String name() default INHERIT_FIELD_NAME;
  
      /**
       * 列的类型，默认为UNDEFINED，其它类型：TEXT、INTEGER、REAL、BLOB
       */
      @SuppressWarnings("unused") @SQLiteTypeAffinity int typeAffinity() default UNDEFINED;
  
      /**
       * 字段是否被索引
       */
      boolean index() default false;
  
      /**
       * 查询结果排序，默认为：UNSPECIFIED
       * BINARY、NOCASE、RTRIM、LOCALIZED、UNICODE
       */
      @Collate int collate() default UNSPECIFIED;
  
      /**
       * 默认值，此默认值通过@Insert插入数据时不会生效，通过@Insert插入数据使用的默认值为属性的默认值
       * 此默认值是通过sql语句插入数据时使用@Query(insert into ...)
       */
      String defaultValue() default VALUE_UNSPECIFIED;
  
      String INHERIT_FIELD_NAME = "[field-name]";
  
      /**
       * 未定义类型
       */
      int UNDEFINED = 1;
      /**
       * 字符串
       */
      int TEXT = 2;
      /**
       * 整型
       */
      int INTEGER = 3;
      /**
       * 浮点或双精度类型(float/double)
       */
      int REAL = 4;
      /**
       * 二进制数据
       */
      int BLOB = 5;
  
      @IntDef({UNDEFINED, TEXT, INTEGER, REAL, BLOB})
      @Retention(RetentionPolicy.CLASS)
      @interface SQLiteTypeAffinity {
      }
  
      /**
       * 索引查询结果排序，按：BINARY处理
       *
       * @see #collate()
       */
      int UNSPECIFIED = 1;
      /**
       * 二进制比较，直接使用memcmp()比较
       *
       * @see #collate()
       */
      int BINARY = 2;
      /**
       * 将26个大写字母转换为小写字母后进行与BINARY一样的比较
       *
       * @see #collate()
       */
      int NOCASE = 3;
      /**
       * 和BINARY一样，忽略结尾的空格
       *
       * @see #collate()
       */
      int RTRIM = 4;
      /**
       * Collation sequence that uses system's current locale.
       *
       * @see #collate()
       */
      @RequiresApi(21)
      int LOCALIZED = 5;
      /**
       * 按Unicode算法排序
       *
       * @see #collate()
       */
      @RequiresApi(21)
      int UNICODE = 6;
  
      @IntDef({UNSPECIFIED, BINARY, NOCASE, RTRIM, LOCALIZED, UNICODE})
      @Retention(RetentionPolicy.CLASS)
      @interface Collate {
      }
  
      String VALUE_UNSPECIFIED = "[value-unspecified]";
  }
  ```
  
* Embedded

  标记一个类为嵌套字段，例如：

  ```kotlin
  @Entity(tableName = ADDRESS_TABLE)
  data class AddressEntity(
      @PrimaryKey
      var code: Int,
      var street: String,
      @Embedded(prefix = "loc_")
      var coordinates: Coordinates?)
  
  data class Coordinates(var latitude: Double, var longitude: Double)
  ```

  在ADDRESS_TABLE表中，会生成四个字段：code、street、loc_latitude、loc_longitude。

  查询结果会以AddressEntity实例反回，属性coordinates的值也会根据loc_latitude、loc_longitude的值构造一个实例，如果这两个值为空，则coordinates变量为空。

  注：Coordinates中的字段可以做为ADDRESS_TABLE表中的主键。

  ```java
  public @interface Embedded {
      /**
       * 为列添加前缀，默认为空
       */
      String prefix() default  "";
  }
  ```

* Database

  标记为数据库类，需要是一个继承`RoomDatabase`的`abstract`类。

  我们创建的数据库一般只有两部分：

  * 一部分是注解数据库信息

    ```java
    @Database(entities = [LocationEntity::class, UserEntity::class], version = 2, exportSchema=false)
    ```

    标记数据库表名和版本号

  * 另一部分是数据库操作行为Dao

    ```java
    @Database(entities = [LocationEntity::class, UserEntity::class], version = 2, exportSchema=false)
    abstract class FMDataBaseRoom: RoomDatabase() {
    
        abstract val locationDao: LocationDao
        abstract val userDao: UserDao
    }
    ```

    

  ```java
  public @interface Database {
      /**
       * 数据库表
       */
      Class<?>[] entities();
  
      /**
       * 数据库视图表
       */
      Class<?>[] views() default {};
  
      /**
       * 数据库版本号
       */
      int version();
  
      /**
       * 默认为true，需要配合注解处理器，在gradle配置room.schemaLocation，把数据库结构以json的形式导出到
       * 配置的目录中，用于数据库迁移
       */
      boolean exportSchema() default true;
  
      /**
       * 数据库自动合并记录，建议每个数据库版本都要有版本升级配置
       * 见 AutoMigration
       * 
       */
      AutoMigration[] autoMigrations() default {};
  }
  ```

* DatabaseView

  ```java
  public @interface DatabaseView {
  
      /**
       * 查询语句（SELECT）
       */
      String value() default "";
  
      /**
       * 视图数据库名子，没有设置默认使用类名
       */
      String viewName() default "";
  }
  ```

  使用

  ```java
  @DatabaseView(
         "SELECT id, name, release_year FROM Song " +
         "WHERE release_year >= 1990 AND release_year <= 1999")
     public class SongFrom90s {
       long id;
       String name;
        @ColumnInfo(name = "release_year")
       private int releaseYear;
     }
  ```

  

* AutoMigration

  Room数据库会自己发现两schema 文件版本的不同，如果是添加表或列，Room会帮我们自动升级版本，如果是删除表、表重命名、删除或列重命名，都需要使用 AutoMigration 操作，其它操作Room会自动帮我们完成。Room做数据库升级时，先把数据拷出来，再重新创建表，再把数据拷进去，每次数据库版本升级后的表结构都与@Entity表相对应。

  需要特别注意的是，在更新表结构时，一定要修改数据库版本号

  每次数据库升级都会产生一条合并记录

  ```java
  public @interface AutoMigration {
      /**
       * 旧版本号
       */
      int from();
  
      /**
       * 新版本号
       */
      int to();
  
      /**
       * 自己实现合并类
       */
      Class<?> spec() default Object.class;
  }
  ```

  

  自己实现操作需要实现接口`AutoMigrationSpec`，在此基本上添加注解进行数据库迁移，迁移操作有：

  * DeleteTable

    ```java
    public @interface DeleteTable {
        /**
         * 要删除的表名
         */
        String tableName();
    
        /**
         * Container annotation for the repeatable annotation {@link DeleteTable}.
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.CLASS)
        @interface Entries {
            DeleteTable[] value();
        }
    }
    ```

    删除表LocationEntity、TestEntity，从数据库版本号1升级到2

    ```java
    @Database(
        entities = [UserEntity::class/*, LocationEntity::class, TestEntity::class*/],
    //    version = 1,
        version = 2,
        exportSchema=true,
        autoMigrations = [
            AutoMigration(from = 1, to = 2, spec = MyMigration::class)
        ]
    )
    abstract class FMDataBaseRoom: RoomDatabase() {...}
    
    
    @DeleteTable.Entries(
        DeleteTable(tableName = LOCATION_TABLE),
        DeleteTable(tableName = TestEntity.TEST_TABLE)
    )
    class MyMigration: AutoMigrationSpec {}
    ```

    

  * RenameTable

    ```java
    @Repeatable(RenameTable.Entries.class)
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.CLASS)
    public @interface RenameTable {
        /**
         * 旧表名
         */
        String fromTableName();
    
        /**
         * 新表名
         */
        String toTableName();
    
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.CLASS)
        @interface Entries {
            RenameTable[] value();
        }
    }
    ```

    数据库表重命名和删除表的方法类似，创建一个AutoMirationSpec的子类，添加注解

    ```java
    @RenameTable(fromTableName = USER_TABLE_OLD, toTableName = USER_TABLE)
    class RenameMigration: AutoMigrationSpec
    ```

    ```java
    @Database(
        entities = [UserEntity::class/*, LocationEntity::class, TestEntity::class*/],
    //    version = 1,
        version = 3,
        exportSchema=true,
        autoMigrations = [
    //        AutoMigration(from = 1, to = 2, spec = MyMigration::class)
            AutoMigration(from = 2, to = 3, spec = RenameMigration::class)
        ]
    )
    abstract class FMDataBaseRoom: RoomDatabase() {...}
    ```

    

  * RenameColumn

    ```java
    public @interface RenameColumn {
        /**
         * 要操作的数据库表名
         */
        String tableName();
    
        /**
         * 要修改的列名
         */
        String fromColumnName();
    
        /**
         * 新的列名
         */
        String toColumnName();
    
        /**
         * 修改多列时使用
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.CLASS)
        @interface Entries {
            RenameColumn[] value();
        }
    }
    ```

    为数据库表中的字段重命名有三个元素：操作的是那个表，旧的字段名，新的字段名

    修改一列

    ```java
    @RenameColumn(tableName = LOCATION_TABLE, fromColumnName = "renameColumn", toColumnName = "renameColumn2")
    class RenameColumnMigration: AutoMigrationSpec
    ```

    修改一列或多列

    ```java
    @RenameColumn.Entries(RenameColumn(tableName = LOCATION_TABLE, fromColumnName = "renameColumn", toColumnName = "renameColumn2"))
    class RenameColumnMigration: AutoMigrationSpec
    ```

    

  * DeleteColumn

    ```java
    public @interface DeleteColumn {
        /**
         * 要操作的表名
         */
        String tableName();
    
        /**
         * 要删除的列名
         */
        String columnName();
    
        /**
         * 删除多列时使用
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.CLASS)
        @interface Entries {
            DeleteColumn[] value();
        }
    }
    ```

    删除一列

    ```java
    @DeleteColumn(tableName = LOCATION_TABLE, columnName = "renameColumn2")
    class DeleteMigration: AutoMigrationSpec
    ```

    删除一列或多列操作见上面其它使用方法

  

  ### 数据库操作

  #### 1.  插入数据：Insert

  ```java
  public @interface Insert {
  
      /**
       * 用于传标记有@Entity的类，默认为方法的参数类
       * 参数只可以是标记为@Entity或者用标记为Entity的集合
       */
      Class<?> entity() default Object.class;
  
      /**
       * 插入数据时遇到冲突解决方式
       * 默认是{@link OnConflictStrategy#ABORT}回滚事务，遇到要插入的数据在数据库中已有则报错
       * {@link OnConflictStrategy#REPLACE}，有则更新无则插入，为insertOrUpdate
       * {@link OnConflictStrategy#IGNORE}，忽略要插入的值
       * 这些操作并不会影响表结构，只是插入数据的操作反馈。更新这些注解时，不需要升级数据库版本
       * 
       */
      @OnConflictStrategy
      int onConflict() default OnConflictStrategy.ABORT;
  }	
  
  
  public @interface OnConflictStrategy {
  
      int REPLACE = 1;
    
      @Deprecated
      int ROLLBACK = 2;
      
      int ABORT = 3;
     
      @Deprecated
      int FAIL = 4;
      
      int IGNORE = 5;
  
  }
  ```

  正常使用只需要在标有@Dao的类中的方法中添加注解就可以

  ```java
  @Dao
  interface LocationDao {
      @Insert
      fun insert(location: LocationEntity)
  }
  ```

  上面的Insert中的entity值会读取方法insert中的LocationEntitiy，如果参数类型没有添加@Entity注解

  ```kotlin
  data class LocationLL(
      //经度
      var longitude: Long = 0,
      //纬度
      var latitude: Long = 0
  )
  
  @Dao
  interface LocationDao {   
      @Insert(entity = LocationEntity::class)
      fun insert(locationLL: LocationLL)
  }
  ```

  #### 2. 删除：Delete

  ```java
  public @interface Delete {
  
      /**
       * 删除方法的实例，使用见Insert
       */
      Class<?> entity() default Object.class;
  }
  ```

  #### 3. 更新：Update

  结构和Insert一样，使用也是一样的，通过主键检查数据是否已存在，存在则更新，不存在则不做处理。

  ```java
  public @interface Update {
  
      Class<?> entity() default Object.class;
  
      
      @OnConflictStrategy
      int onConflict() default OnConflictStrategy.ABORT;
  }
  ```

  #### 4. 查询：Query

  ```java
  public @interface Query {
      /**
       * sql语句
       */
      String value();
  }
  ```

  使用：

  ```java
  @Query("SELECT * FROM $ADDRESS_TABLE WHERE code= :code")
  fun selectAddress(code: Int): AddressEntity
  ```

  也可以是insert、update、delete操作语句

  #### 5. 事务：Transaction

  事务用在Dao抽象类中的非抽象方法上，或者用于java8的Dao接口默认实现方法中，在这两种方法中可以实现自己的操作，也可以在@Query方法上标记，这样只对当前sql生效。

  #### 6. 类型转换器：TypeConverter

  TypeConverter添加在要实现类型转换类的方法上

  ```java
  class TimeConverter {
      @TypeConverter
      fun fromTimestamp(value: Long?): Date? {
          return value?.let { Date(it) }
      }
  
      @TypeConverter
      fun dateToTimestamp(date: Date?): Long? {
          return date?.time
      }
  }
  ```

  然后把此转换类添加到标有@Entity类上，或着数据库上@Database，源码中解释可以添加到字段或方法中，经测试，添加字段、方法或着@Dao上无效，也可能是我操作的方法不对，@TypeConverter标记在转换类上，@TypeConverters用于标记在@Entity或@database上，标记使用类型转换例如：

  ```java
  @Entity(
      tableName = LOCATION_TABLE,
  //    foreignKeys = [ForeignKey(
  //        entity = UserEntity::class,
  //        parentColumns = ["userId"],
  //        childColumns = ["userId"],
  //        onDelete = ForeignKey.SET_DEFAULT,
  //    )]
  )
  @TypeConverters(TimeConverter::class)
  data class LocationEntity(...)
  ```

  

## Room源码分析

我们先来看一Room的几个核心类，通过这几个类，我们可以了解到数据库该怎么创建，有特殊需求时该怎么配置，还有就是Room是怎么实现数据映射到具体实体类中的。

### Room类

Room类是一个工具类，对我们开放的两个方法`databaseBuilder`和`inMemoryDatabaseBuilder`，根据我们的用途来使用不同的数据库。

```java
public class Room {
  /**
   *	创建持久化数据实例，使用Room不能能过构造方法来创建
   **/
  public static <T extends RoomDatabase> RoomDatabase.Builder<T> databaseBuilder(
            @NonNull Context context, @NonNull Class<T> klass, @NonNull String name) {...}
  
  /**
   *	创建内存数据库，数据存储在内存中，当进行被杀死，数据就丢失。
   **/
  public static <T extends RoomDatabase> RoomDatabase.Builder<T> inMemoryDatabaseBuilder(
            @NonNull Context context, @NonNull Class<T> klass) {
        return new RoomDatabase.Builder<>(context, klass, null);
    }
  
  /**
   *	通过反射生成数据库类对象，在RoomDatabase类中会回调回来
   *
   **/
  public static <T, C> T getGeneratedImplementation(@NonNull Class<C> klass,
            @NonNull String suffix) {
    final String fullPackage = klass.getPackage().getName();
        String name = klass.getCanonicalName();
        final String postPackageName = fullPackage.isEmpty()
                ? name
                : name.substring(fullPackage.length() + 1);
        final String implName = postPackageName.replace('.', '_') + suffix;
        //noinspection TryWithIdenticalCatches
        try {

            final String fullClassName = fullPackage.isEmpty()
                    ? implName
                    : fullPackage + "." + implName;
            @SuppressWarnings("unchecked")
            final Class<T> aClass = (Class<T>) Class.forName(
                    fullClassName, true, klass.getClassLoader());
            return aClass.newInstance();
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("cannot find implementation for "
                    + klass.getCanonicalName() + ". " + implName + " does not exist");
        } catch (IllegalAccessException e) {
            throw new RuntimeException("Cannot access the constructor"
                    + klass.getCanonicalName());
        } catch (InstantiationException e) {
            throw new RuntimeException("Failed to create an instance of "
                    + klass.getCanonicalName());
        }
  }
}
```

通过这两个方法可以发现，创建什么类型的数据库，区别是否传了数据库名称，如果没有传则是内存型数据库，传了则是持久型数据库

### RoomDatabase类

这是数据库的核心类，我们自定义的数据库类也是需要实现这个抽象类的

```java
@Database(
    entities = [UserEntity::class, LocationEntity::class, AddressEntity::class/*, TestEntity::class*/],
//    version = 1,
    version = 18,
    exportSchema=true,
    autoMigrations = [
        AutoMigration(from = 1, to = 2, spec = MyMigration::class),
        AutoMigration(from = 2, to = 3, spec = RenameMigration::class),
        AutoMigration(from = 9, to = 10, spec = RenameColumnMigration::class),
        AutoMigration(from = 11, to = 12, spec = DeleteMigration::class),
    ]
)
//@TypeConverters(TimeConverter::class)
abstract class FMDataBaseRoom: RoomDatabase() {
  //TODO Dao抽象方法
  //TODO 数据库单例获取
  //TODO ...
}


```

RoomDatabase类不小，但内容基本上都是一些数据配置，真正的数据库操作在APT自动生成的实现类中：`xxx_Impl`

#### RoomDatabase配置

​		RoomDatabase类中配置信息有两部分是重复的，一是Builder构建者模式的配置信息，另一部分是类本身的配置信息，就以`Builder`类中的信息讲解

```java
public abstract class RoomDatabase {
  ...
  public static class Builder<T extends RoomDatabase> {
    //数据库类
    private final Class<T> mDatabaseClass;
    //数据库名称，如果是内存数据库，则此值为null
    private final String mName;
    //此值是长持有，需要传application的Context
    private final Context mContext;
    //数据库操作回调：创建数据库、打开数据库、销毁数据库，这三种操作都会触发回调对应的方法，调用地方在数据库实现类中。
    private ArrayList<Callback> mCallbacks;
    //当数据库从其它文件中拷进来时（createFromAsset、createFromFile、createFromInputStream），调用此回调
    private PrepackagedDatabaseCallback mPrepackagedDatabaseCallback;
    //当查询语句被执行时，调用此回调，虽然添加此回调开销很小，但建议非必要的情况下不使用。
    private QueryCallback mQueryCallback;
    //查询语句回调线程池
    private Executor mQueryCallbackExecutor;
    //类型转换集合
    private List<Object> mTypeConverters;
    //数据库迁移操作描述集合
    private List<AutoMigrationSpec> mAutoMigrationSpecs;

    //数据查询线程池
    private Executor mQueryExecutor;
    //事务执行线程池
    private Executor mTransactionExecutor;
    //创建SupportSQLiteOpenHelper对象工厂类
    private SupportSQLiteOpenHelper.Factory mFactory;
    //是否允许在主线程中执行sql语句
    private boolean mAllowMainThreadQueries;
    //日志模型
    private JournalMode mJournalMode;
    //数据库表发生变化时通过其它数据库实例，前提是所有实例必须使用同一个数据库文件，否则也没有意义。
    private Intent mMultiInstanceInvalidationIntent;
    //是否允许数据库升级重新建新库
    private boolean mRequireMigration;
    //
    private boolean mAllowDestructiveMigrationOnDowngrade;
		//在数据库空闲多长时间关闭数据库
    private long mAutoCloseTimeout = -1L;
    //数据库空闲时间单位
    private TimeUnit mAutoCloseTimeUnit;

    //数据库迁移容器集合
    private final MigrationContainer mMigrationContainer;
    //重指定迁移版本开始重新创建数据库表
    private Set<Integer> mMigrationsNotRequiredFrom;
    //数据库迁移版本号集合
    private Set<Integer> mMigrationStartAndEndVersions;
		//Asset路径，数据库根据此路径下的数据库备份生成
    private String mCopyFromAssetPath;
    //根据此文件数据库备份，生成数据库
    private File mCopyFromFile;
    //根据数据流生成数据库
    private Callable<InputStream> mCopyFromInputStream;
   
  ...
}
```

执行Builder中的build()时，主要会做两个操作，一个是创建sql查询和事务线程池，另一个是创建工厂类：基本工厂、自动关闭数据操作工厂、数据库从文件生成工厂，代码如下：

```java
/**
 *	初始化数据库类
 **/
public void init(@NonNull DatabaseConfiguration configuration) {
  	//创建数据库帮助类，这是我们操作数据库的核心类，
  	//createOpenHelper()方法会在我们自定义的数据库类标有@Database的子类中生成此方法的实现。
    mOpenHelper = createOpenHelper(configuration);
    Set<Class<? extends AutoMigrationSpec>> requiredAutoMigrationSpecs =
            getRequiredAutoMigrationSpecs();
    BitSet usedSpecs = new BitSet();
    for (Class<? extends AutoMigrationSpec> spec : requiredAutoMigrationSpecs) {
        int foundIndex = -1;
        for (int providedIndex = configuration.autoMigrationSpecs.size() - 1;
                providedIndex >= 0; providedIndex--
        ) {
            Object provided = configuration.autoMigrationSpecs.get(providedIndex);
            if (spec.isAssignableFrom(provided.getClass())) {
                foundIndex = providedIndex;
                usedSpecs.set(foundIndex);
                break;
            }
        }
        if (foundIndex < 0) {
            throw new IllegalArgumentException(
                    "A required auto migration spec (" + spec.getCanonicalName()
                            + ") is missing in the database configuration.");
        }
        mAutoMigrationSpecs.put(spec, configuration.autoMigrationSpecs.get(foundIndex));
    }

    for (int providedIndex = configuration.autoMigrationSpecs.size() - 1;
            providedIndex >= 0; providedIndex--) {
        if (!usedSpecs.get(providedIndex)) {
            throw new IllegalArgumentException("Unexpected auto migration specs found. "
                    + "Annotate AutoMigrationSpec implementation with "
                    + "@ProvidedAutoMigrationSpec annotation or remove this spec from the "
                    + "builder.");
        }
    }
		
  	//新的版本迁移合并到版本迁移容器中
    List<Migration> autoMigrations = getAutoMigrations(mAutoMigrationSpecs);
    for (Migration autoMigration : autoMigrations) {
        boolean migrationExists = configuration.migrationContainer.getMigrations()
                        .containsKey(autoMigration.startVersion);
        if (!migrationExists) {
            configuration.migrationContainer.addMigrations(autoMigration);
        }
    }

    // 配置 SqliteCopyOpenHelper
    SQLiteCopyOpenHelper copyOpenHelper = unwrapOpenHelper(SQLiteCopyOpenHelper.class,
            mOpenHelper);
    if (copyOpenHelper != null) {
        copyOpenHelper.setDatabaseConfiguration(configuration);
    }
		//配置 AutoClosingRoomOpenHelper
    AutoClosingRoomOpenHelper autoClosingRoomOpenHelper =
            unwrapOpenHelper(AutoClosingRoomOpenHelper.class, mOpenHelper);

    if (autoClosingRoomOpenHelper != null) {
        mAutoCloser = autoClosingRoomOpenHelper.getAutoCloser();
        mInvalidationTracker.setAutoCloser(mAutoCloser);
    }

		//设置数据库日志是否可用
    boolean wal = false;
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
        wal = configuration.journalMode == JournalMode.WRITE_AHEAD_LOGGING;
        mOpenHelper.setWriteAheadLoggingEnabled(wal);
    }
  	//设置数据库基本变量
    mCallbacks = configuration.callbacks;
    mQueryExecutor = configuration.queryExecutor;
    mTransactionExecutor = new TransactionExecutor(configuration.transactionExecutor);
    mAllowMainThreadQueries = configuration.allowMainThreadQueries;
    mWriteAheadLoggingEnabled = wal;
    if (configuration.multiInstanceInvalidationServiceIntent != null) {
        mInvalidationTracker.startMultiInstanceInvalidation(configuration.context,
                configuration.name, configuration.multiInstanceInvalidationServiceIntent);
    }

    Map<Class<?>, List<Class<?>>> requiredFactories = getRequiredTypeConverters();
    // 检查类型转换器是否可用，并转存到类变量中
    BitSet used = new BitSet();
    for (Map.Entry<Class<?>, List<Class<?>>> entry : requiredFactories.entrySet()) {
        Class<?> daoName = entry.getKey();
        for (Class<?> converter : entry.getValue()) {
            int foundIndex = -1;
            // traverse provided converters in reverse so that newer one overrides
            for (int providedIndex = configuration.typeConverters.size() - 1;
                    providedIndex >= 0; providedIndex--) {
                Object provided = configuration.typeConverters.get(providedIndex);
                if (converter.isAssignableFrom(provided.getClass())) {
                    foundIndex = providedIndex;
                    used.set(foundIndex);
                    break;
                }
            }
            if (foundIndex < 0) {
                throw new IllegalArgumentException(
                        "A required type converter (" + converter + ") for"
                                + " " + daoName.getCanonicalName()
                                + " is missing in the database configuration.");
            }
            mTypeConverters.put(converter, configuration.typeConverters.get(foundIndex));
        }
    }
    // 检查提供的factory类是否可用
    for (int providedIndex = configuration.typeConverters.size() - 1;
            providedIndex >= 0; providedIndex--) {
        if (!used.get(providedIndex)) {
            Object converter = configuration.typeConverters.get(providedIndex);
            throw new IllegalArgumentException("Unexpected type converter " + converter + ". "
                    + "Annotate TypeConverter class with @ProvidedTypeConverter annotation "
                    + "or remove this converter from the builder.");
        }
    }
}
```

另外提供几个抽象方法，在子类中自动实现

```java
public final class FMDataBaseRoom_Impl extends FMDataBaseRoom {
  //创建OpenHelper，
  @Override
  protected SupportSQLiteOpenHelper createOpenHelper(DatabaseConfiguration configuration) {
    //此方法为自动实现，主要实现了SupportSQLiteOpenHelper#Callback接口。
  }
  //清空数据库表数据
  public abstract void clearAllTables();
  //数据库是否已打开
  public boolean isOpen();
  //关闭数据库，如果已打开
  public void close();
  //查询数据库方法
  public Cursor query(...);
  ...
    
}
```

RoomDatabase抽象类主要是持有一些数据库常用变量及方法，为子类自动实现提供数据。

RoomDatabase实现类是自动生成的，对数据库的创建删除数据、删除表等操作都是由Room帮我们根据添加的注解自动生成。

接下来我们看下表操作

#### Dao实现现类：xxxDao_Impl

以我LocationDao为例；

每个对象操作都会对应一个适配器，两个对象插入操作会生成两个适配器，

适配器分为两部分：sql语句、和数据邦定`bind`，在Dao中创建的操作方法，会生成相应的对应方法，添加数据库的相应保护及数据转换

请看下面类中的方法实现。

```java
//我们写的抽象类或接口
@Dao
@TypeConverters(TimeConverter::class)
interface LocationDao {
    @Insert
    @TypeConverters(TimeConverter::class)
    fun insert(location: LocationEntity)

    @Insert(entity = LocationEntity::class)
    fun insert(locationLL: LocationLL)

    @Query("insert into $LOCATION_TABLE ('longitude', 'latitude') values (1, 1)")
    fun insert()

    @Delete
    fun delete(location: LocationEntity)

    @Query("SELECT * FROM $LOCATION_TABLE WHERE id= :id")
    fun getId(id: Int): LocationEntity?

    @Query("SELECT * FROM $LOCATION_TABLE WHERE id= :id")
    fun getIdByLiveData(id: Int): LiveData<LocationEntity?>

    @Update(onConflict = OnConflictStrategy.REPLACE)
    fun updateLocation(location: LocationEntity): Int

    @Query("SELECT * FROM $LOCATION_TABLE WHERE id= :id")
    fun getCoordinates(id: Int): Coordinates

    @Transaction
    fun insertAndDelete(newLocation: LocationEntity, newLocation2: LocationEntity, oldLocationEntity: 	LocationEntity) {
        insert(newLocation)
        insert(newLocation2)
        delete(oldLocationEntity)
    }
}
```

Dao实现类

```java
/**
 *	Dao实现类
 **/
public final class LocationDao_Impl implements LocationDao {
  //数据库对象
  private final RoomDatabase __db;
	//实例LocationEntity的Insert适配器
  private final EntityInsertionAdapter<LocationEntity> __insertionAdapterOfLocationEntity;
	//类型转换器，在适配器中会使用类型转换器的方法，在查询的方法也会使用反转方法。
  private final TimeConverter __timeConverter = new TimeConverter();
	//实例LocationLL的Insert适配器
  private final EntityInsertionAdapter<LocationLL> __insertionAdapterOfLocationLLAsLocationEntity;
	//LocationEntity的delete适配器
  private final EntityDeletionOrUpdateAdapter<LocationEntity> __deletionAdapterOfLocationEntity;
	//LocationEntity的update适配器
  private final EntityDeletionOrUpdateAdapter<LocationEntity> __updateAdapterOfLocationEntity;
	//Sql操作控制类，见下面对此类的解释
  private final SharedSQLiteStatement __preparedStmtOfInsert;

  public LocationDao_Impl(RoomDatabase __db) {
    this.__db = __db;
    //创建相应的适配器
    this.__insertionAdapterOfLocationEntity = new EntityInsertionAdapter<LocationEntity>(__db) {
      //自动生成的创建SQL
      @Override
      public String createQuery() {
        return "INSERT OR ABORT INTO `location_table` (`id`,`longitude`,`latitude`,`userId`,`timestamp`) VALUES (nullif(?, 0),?,?,?,?)";
      }
			
      //数据邦定
      @Override
      public void bind(SupportSQLiteStatement stmt, LocationEntity value) {
        stmt.bindLong(1, value.getId());
        stmt.bindLong(2, value.getLongitude());
        stmt.bindLong(3, value.getLatitude());
        if (value.getUserId() == null) {
          stmt.bindNull(4);
        } else {
          stmt.bindLong(4, value.getUserId());
        }
        //类型转换器，Date转Long
        final Long _tmp = __timeConverter.dateToTimestamp(value.getTimestamp());
        if (_tmp == null) {
          stmt.bindNull(5);
        } else {
          stmt.bindLong(5, _tmp);
        }
      }
    };
    //创建相应的适配器2
    this.__insertionAdapterOfLocationLLAsLocationEntity = new EntityInsertionAdapter<LocationLL>(__db) {
      @Override
      public String createQuery() {
        return "INSERT OR ABORT INTO `location_table` (`longitude`,`latitude`) VALUES (?,?)";
      }

      @Override
      public void bind(SupportSQLiteStatement stmt, LocationLL value) {
        stmt.bindLong(1, value.getLongitude());
        stmt.bindLong(2, value.getLatitude());
      }
    };
    //创建相应的适配器3
    this.__deletionAdapterOfLocationEntity = new EntityDeletionOrUpdateAdapter<LocationEntity>(__db) {
      @Override
      public String createQuery() {
        return "DELETE FROM `location_table` WHERE `id` = ?";
      }

      @Override
      public void bind(SupportSQLiteStatement stmt, LocationEntity value) {
        stmt.bindLong(1, value.getId());
      }
    };
    //创建相应的适配器4
    this.__updateAdapterOfLocationEntity = new EntityDeletionOrUpdateAdapter<LocationEntity>(__db) {
      @Override
      public String createQuery() {
        return "UPDATE OR REPLACE `location_table` SET `id` = ?,`longitude` = ?,`latitude` = ?,`userId` = ?,`timestamp` = ? WHERE `id` = ?";
      }

      @Override
      public void bind(SupportSQLiteStatement stmt, LocationEntity value) {
        stmt.bindLong(1, value.getId());
        stmt.bindLong(2, value.getLongitude());
        stmt.bindLong(3, value.getLatitude());
        if (value.getUserId() == null) {
          stmt.bindNull(4);
        } else {
          stmt.bindLong(4, value.getUserId());
        }
        final Long _tmp = __timeConverter.dateToTimestamp(value.getTimestamp());
        if (_tmp == null) {
          stmt.bindNull(5);
        } else {
          stmt.bindLong(5, _tmp);
        }
        stmt.bindLong(6, value.getId());
      }
    };
    //直接写死的sql，直接使用SharedSQLiteStatement，免去了适配器操作，创建多个时会生成__preparedStmtOfInsert2等变量
    this.__preparedStmtOfInsert = new SharedSQLiteStatement(__db) {
      @Override
      public String createQuery() {
        final String _query = "insert into location_table ('longitude', 'latitude') values (1, 1)";
        return _query;
      }
    };
  }

  /**
   *	在事务中插入数据，适配器首通会通过构造方法中创建的bin()把数据赋值给stmt然后提交操作把数据插入数据库，
   **/
  @Override
  public void insert(final LocationEntity location) {
    __db.assertNotSuspendingTransaction();
    __db.beginTransaction();
    try {
      __insertionAdapterOfLocationEntity.insert(location);
      __db.setTransactionSuccessful();
    } finally {
      __db.endTransaction();
    }
  }

  @Override
  public void insert(final LocationLL locationLL) {
    __db.assertNotSuspendingTransaction();
    __db.beginTransaction();
    try {
      __insertionAdapterOfLocationLLAsLocationEntity.insert(locationLL);
      __db.setTransactionSuccessful();
    } finally {
      __db.endTransaction();
    }
  }

  @Override
  public void delete(final LocationEntity location) {
    __db.assertNotSuspendingTransaction();
    __db.beginTransaction();
    try {
      __deletionAdapterOfLocationEntity.handle(location);
      __db.setTransactionSuccessful();
    } finally {
      __db.endTransaction();
    }
  }

  @Override
  public int updateLocation(final LocationEntity location) {
    __db.assertNotSuspendingTransaction();
    int _total = 0;
    __db.beginTransaction();
    try {
      _total +=__updateAdapterOfLocationEntity.handle(location);
      __db.setTransactionSuccessful();
      return _total;
    } finally {
      __db.endTransaction();
    }
  }

  /**
   *	此方法是在接口中定义为事务的方法，这样在处理操作调用的是接口默认类
   *
   **/
  @Override
  public void insertAndDelete(final LocationEntity newLocation, final LocationEntity newLocation2,
      final LocationEntity oldLocationEntity) {
    __db.beginTransaction();
    try {
      LocationDao.DefaultImpls.insertAndDelete(LocationDao_Impl.this, newLocation, newLocation2, oldLocationEntity);
      __db.setTransactionSuccessful();
    } finally {
      __db.endTransaction();
    }
  }

  /**
   *	此处的处理省去了适配器的操作，直接传入sql
   **/
  @Override
  public void insert() {
    __db.assertNotSuspendingTransaction();
    final SupportSQLiteStatement _stmt = __preparedStmtOfInsert.acquire();
    __db.beginTransaction();
    try {
      _stmt.executeInsert();
      __db.setTransactionSuccessful();
    } finally {
      __db.endTransaction();
      __preparedStmtOfInsert.release(_stmt);
    }
  }

  /**
   *	这里的代码比较多，原理和上面一样，首先确定一个sql操作语句，邦定动态参数到sql中，通过条件查询到数据对象Cursor
   *	然后根据数据生成要返回的对象。这些以前是我们手动写的，现在Room帮我们生成好了。
   *
   **/
  @Override
  public LocationEntity getId(final int id) {
    final String _sql = "SELECT * FROM location_table WHERE id= ?";
    final RoomSQLiteQuery _statement = RoomSQLiteQuery.acquire(_sql, 1);
    int _argIndex = 1;
    _statement.bindLong(_argIndex, id);
    __db.assertNotSuspendingTransaction();
    final Cursor _cursor = DBUtil.query(__db, _statement, false, null);
    try {
      final int _cursorIndexOfId = CursorUtil.getColumnIndexOrThrow(_cursor, "id");
      final int _cursorIndexOfLongitude = CursorUtil.getColumnIndexOrThrow(_cursor, "longitude");
      final int _cursorIndexOfLatitude = CursorUtil.getColumnIndexOrThrow(_cursor, "latitude");
      final int _cursorIndexOfUserId = CursorUtil.getColumnIndexOrThrow(_cursor, "userId");
      final int _cursorIndexOfTimestamp = CursorUtil.getColumnIndexOrThrow(_cursor, "timestamp");
      final LocationEntity _result;
      if(_cursor.moveToFirst()) {
        final int _tmpId;
        _tmpId = _cursor.getInt(_cursorIndexOfId);
        final long _tmpLongitude;
        _tmpLongitude = _cursor.getLong(_cursorIndexOfLongitude);
        final long _tmpLatitude;
        _tmpLatitude = _cursor.getLong(_cursorIndexOfLatitude);
        final Integer _tmpUserId;
        if (_cursor.isNull(_cursorIndexOfUserId)) {
          _tmpUserId = null;
        } else {
          _tmpUserId = _cursor.getInt(_cursorIndexOfUserId);
        }
        final Date _tmpTimestamp;
        final Long _tmp;
        if (_cursor.isNull(_cursorIndexOfTimestamp)) {
          _tmp = null;
        } else {
          _tmp = _cursor.getLong(_cursorIndexOfTimestamp);
        }
        _tmpTimestamp = __timeConverter.fromTimestamp(_tmp);
        _result = new LocationEntity(_tmpId,_tmpLongitude,_tmpLatitude,_tmpUserId,_tmpTimestamp);
      } else {
        _result = null;
      }
      return _result;
    } finally {
      _cursor.close();
      _statement.release();
    }
  }

  /**
   *	通过生命周期组件获取查询结果
   *	LiveData的使用或RxJava等生命周期组件也是Room核心的知识点。
   **/
  @Override
  public LiveData<LocationEntity> getIdByLiveData(final int id) {
    final String _sql = "SELECT * FROM location_table WHERE id= ?";
    final RoomSQLiteQuery _statement = RoomSQLiteQuery.acquire(_sql, 1);
    int _argIndex = 1;
    _statement.bindLong(_argIndex, id);
    //创建一个LiveData对象，在需要操作数据时，会调用回调方法call()，通过call()得到查询结果，再把结果发送给LiveData
    return __db.getInvalidationTracker().createLiveData(new String[]{"location_table"}, false, new Callable<LocationEntity>() {
      @Override
      public LocationEntity call() throws Exception {
        //查询数据库，并把结果映射到LocationEntity类中返回
        final Cursor _cursor = DBUtil.query(__db, _statement, false, null);
        try {
          final int _cursorIndexOfId = CursorUtil.getColumnIndexOrThrow(_cursor, "id");
          final int _cursorIndexOfLongitude = CursorUtil.getColumnIndexOrThrow(_cursor, "longitude");
          final int _cursorIndexOfLatitude = CursorUtil.getColumnIndexOrThrow(_cursor, "latitude");
          final int _cursorIndexOfUserId = CursorUtil.getColumnIndexOrThrow(_cursor, "userId");
          final int _cursorIndexOfTimestamp = CursorUtil.getColumnIndexOrThrow(_cursor, "timestamp");
          final LocationEntity _result;
          if(_cursor.moveToFirst()) {
            final int _tmpId;
            _tmpId = _cursor.getInt(_cursorIndexOfId);
            final long _tmpLongitude;
            _tmpLongitude = _cursor.getLong(_cursorIndexOfLongitude);
            final long _tmpLatitude;
            _tmpLatitude = _cursor.getLong(_cursorIndexOfLatitude);
            final Integer _tmpUserId;
            if (_cursor.isNull(_cursorIndexOfUserId)) {
              _tmpUserId = null;
            } else {
              _tmpUserId = _cursor.getInt(_cursorIndexOfUserId);
            }
            final Date _tmpTimestamp;
            final Long _tmp;
            if (_cursor.isNull(_cursorIndexOfTimestamp)) {
              _tmp = null;
            } else {
              _tmp = _cursor.getLong(_cursorIndexOfTimestamp);
            }
            _tmpTimestamp = __timeConverter.fromTimestamp(_tmp);
            _result = new LocationEntity(_tmpId,_tmpLongitude,_tmpLatitude,_tmpUserId,_tmpTimestamp);
          } else {
            _result = null;
          }
          return _result;
        } finally {
          _cursor.close();
        }
      }

      @Override
      protected void finalize() {
        _statement.release();
      }
    });
  }

  @Override
  public Coordinates getCoordinates(final int id) {
    final String _sql = "SELECT * FROM location_table WHERE id= ?";
    final RoomSQLiteQuery _statement = RoomSQLiteQuery.acquire(_sql, 1);
    int _argIndex = 1;
    _statement.bindLong(_argIndex, id);
    __db.assertNotSuspendingTransaction();
    final Cursor _cursor = DBUtil.query(__db, _statement, false, null);
    try {
      final int _cursorIndexOfLongitude = CursorUtil.getColumnIndexOrThrow(_cursor, "longitude");
      final int _cursorIndexOfLatitude = CursorUtil.getColumnIndexOrThrow(_cursor, "latitude");
      final Coordinates _result;
      if(_cursor.moveToFirst()) {
        final double _tmpLongitude;
        _tmpLongitude = _cursor.getDouble(_cursorIndexOfLongitude);
        final double _tmpLatitude;
        _tmpLatitude = _cursor.getDouble(_cursorIndexOfLatitude);
        _result = new Coordinates(_tmpLatitude,_tmpLongitude);
      } else {
        _result = null;
      }
      return _result;
    } finally {
      _cursor.close();
      _statement.release();
    }
  }

  public static List<Class<?>> getRequiredConverters() {
    return Collections.emptyList();
  }
}


```

#### EntityInsertionAdapter

以`EntityInsertionAdapter`、`EntityDeletionOrUpdateAdapter`功能类似，都是增加了一个抽象方法bind()，添加相应的操作方法，下面以`EntityInsertionAdapter`为例：

```java
public abstract class EntityInsertionAdapter<T> extends SharedSQLiteStatement {
  ...
  protected abstract void bind(SupportSQLiteStatement statement, T entity);
  
  //插入一个对象
   public final void insert(T entity) {
        final SupportSQLiteStatement stmt = acquire();
        try {
            bind(stmt, entity);
            stmt.executeInsert();
        } finally {
            release(stmt);
        }
    }

    //插入对象数组
    public final void insert(T[] entities) {
        final SupportSQLiteStatement stmt = acquire();
        try {
            for (T entity : entities) {
                bind(stmt, entity);
                stmt.executeInsert();
            }
        } finally {
            release(stmt);
        }
    }
  ...
}

public abstract class SharedSQLiteStatement {
    private final AtomicBoolean mLock = new AtomicBoolean(false);

    private final RoomDatabase mDatabase;
    private volatile SupportSQLiteStatement mStmt;

    /**
     * Creates an SQLite prepared statement that can be re-used across threads. If it is in use,
     * it automatically creates a new one.
     *
     * @param database The database to create the statement in.
     */
    public SharedSQLiteStatement(RoomDatabase database) {
        mDatabase = database;
    }

    //创建要操作的sql语句
    protected abstract String createQuery();

    protected void assertNotMainThread() {
        mDatabase.assertNotMainThread();
    }

  	//创建操作实例
    private SupportSQLiteStatement createNewStatement() {
        String query = createQuery();
        return mDatabase.compileStatement(query);
    }

  	// 获取操作实例
    private SupportSQLiteStatement getStmt(boolean canUseCached) {
        final SupportSQLiteStatement stmt;
        if (canUseCached) {
            if (mStmt == null) {
                mStmt = createNewStatement();
            }
            stmt = mStmt;
        } else {
            stmt = createNewStatement();
        }
        return stmt;
    }

    /**
     * 获取操作实例
     */
    public SupportSQLiteStatement acquire() {
        assertNotMainThread();
        return getStmt(mLock.compareAndSet(false, true));
    }

    //操作释放
    public void release(SupportSQLiteStatement statement) {
        if (statement == mStmt) {
            mLock.set(false);
        }
    }
}

public interface SupportSQLiteStatement extends SupportSQLiteProgram {
    /**
     * 执行SQL语句
     */
    void execute();

    /**
     * 执行SQL更新或删除操作，返回操作影响的数据条数
     */
    int executeUpdateDelete();

    /**
     * 执行插入操作，如果成功返回最后一条记录的行ID，如果不成功则返回-1
     */
    long executeInsert();

    /**
     * 执行查询语句，返回结果条数
     */
    long simpleQueryForLong();
    /**
     * 执行查询语句，返回结果条数
     */
    String simpleQueryForString();
}

/**
 *	sql赋值操作
 **/
public interface SupportSQLiteProgram extends Closeable {
    
    void bindNull(int index);

    void bindLong(int index, long value);

  	void bindDouble(int index, double value);

    void bindString(int index, String value);

  	void bindBlob(int index, byte[] value);

    void clearBindings();
}
```

​		上面的一些类就是Room实现数据操作的流程。理解也很简单，整体流程是：通这注解@Entity标注出数据库表信息，通过@Dao标注对数据库的操作，通过@Database标出数据库的信息。Room通过APT在编译时生成我们使用以前手写的代码。比如创建数据库表，实现Dao类，完成对数据库的操作。

​		接下来我们看下Room的观察者，实现数据有变化自动更新查询结果

## LiveData在Room中使用

Room可以实现LiveData、RxJava、Kotlin协程等观察者模式，实现数据有变化时，可以动态刷新数据到UI层，我们以LiveData为例了解下Room是怎么样实现的，首先我们先了解下几个类，然后看下整体的流程是什么，

接下来的类在`room-runtime`库中

首先看下`RoomTrackingLiveData`,继承LiveData

在LiveData处于活跃时，实现数据库驱动Livedata查询数据，核心的是mRefreshRunnable变量，在数据有变化时会调用刷新Runnable来查询数据，结果通过postValue通过UI层来刷新页面。

注意：当表数据有变化时就会收到通知，添加过注册监听的方法就会重新查询数据，这里有一个问题时，如果要查询的数据并非全量数据，可能会导致做无用功，如果是分页查询，当前查询到了最后一页，但更新的数据是第一条，这样重新查询真实的数据变化并不会反馈到UI层，或者导致多一次无用的刷新页面；还有一种情况就是查询的是全量数据，但更新的字段并不会影响UI层显示，这样也是不用操作。建议最好在使用监听数据变化更新UI时，判断当前操作是否需要监听数据变化，逻辑层还要添加数据比较功能，这样当数据变化后，如果不影响UI展示或逻辑处理，在逻辑层就过滤掉此操作向下传递来更新UI。

```java
class RoomTrackingLiveData<T> extends LiveData<T> {
    @SuppressWarnings("WeakerAccess")
    final RoomDatabase mDatabase;

    @SuppressWarnings("WeakerAccess")
    final boolean mInTransaction;

    @SuppressWarnings("WeakerAccess")
    final Callable<T> mComputeFunction;

    private final InvalidationLiveDataContainer mContainer;

    @SuppressWarnings("WeakerAccess")
    final InvalidationTracker.Observer mObserver;

    @SuppressWarnings("WeakerAccess")
    final AtomicBoolean mInvalid = new AtomicBoolean(true);

    @SuppressWarnings("WeakerAccess")
    final AtomicBoolean mComputing = new AtomicBoolean(false);

    @SuppressWarnings("WeakerAccess")
    final AtomicBoolean mRegisteredObserver = new AtomicBoolean(false);

  	//当页面从非活跃切换到活跃状态时，或着当前状态为活跃且数据有变化时，会调用此任务重新查表
    @SuppressWarnings("WeakerAccess")
    final Runnable mRefreshRunnable = new Runnable() {
        @WorkerThread
        @Override
        public void run() {
          	//判断是否已添加过观察者，如果没有添加则向InvalidationTracker添加观察者
          	//如果数据有变化则调用mObserver.onInvalidated(Set<String>）来重新查询数据。
            if (mRegisteredObserver.compareAndSet(false, true)) {
                mDatabase.getInvalidationTracker().addWeakObserver(mObserver);
            }
            boolean computed;
            do {
                computed = false;
                // 过虑重复操作，条件执行完成会把状态再切回来
                if (mComputing.compareAndSet(false, true)) {
                    try {
                        T value = null;
                      	//数据是否有变化，如果有则进入循环，查询数据
                        while (mInvalid.compareAndSet(true, false)) {
                            computed = true;
                            try {
                                value = mComputeFunction.call();
                            } catch (Exception e) {
                                throw new RuntimeException("Exception while computing database"
                                        + " live data.", e);
                            }
                        }
                      	//如果有新数据变化，则更新LiveData数据
                        if (computed) {
                            postValue(value);
                        }
                    } finally {
												//释放锁
                        mComputing.set(false);
                    }
                }
                //前提是有数据变化
            } while (computed && mInvalid.get());
        }
    };

  	//数据有变化运行此方法
    @SuppressWarnings("WeakerAccess")
    final Runnable mInvalidationRunnable = new Runnable() {
        @MainThread
        @Override
        public void run() {
            boolean isActive = hasActiveObservers();
            if (mInvalid.compareAndSet(false, true)) {
                if (isActive) {
                    getQueryExecutor().execute(mRefreshRunnable);
                }
            }
        }
    };
    @SuppressLint("RestrictedApi")
    RoomTrackingLiveData(
            RoomDatabase database,
            InvalidationLiveDataContainer container,
            boolean inTransaction,
            Callable<T> computeFunction,
            String[] tableNames) {
        mDatabase = database;
        mInTransaction = inTransaction;
        mComputeFunction = computeFunction;
        mContainer = container;
      	//监听数据变化
        mObserver = new InvalidationTracker.Observer(tableNames) {
            @Override
            public void onInvalidated(@NonNull Set<String> tables) {
                ArchTaskExecutor.getInstance().executeOnMainThread(mInvalidationRunnable);
            }
        };
    }

    @Override
    protected void onActive() {
        super.onActive();
      	//处于活跃时，把LiveData加入到LiveData容器中
        mContainer.onActive(this);
        getQueryExecutor().execute(mRefreshRunnable);
    }

    @Override
    protected void onInactive() {
        super.onInactive();
      	//从窗口中移除LiveData
        mContainer.onInactive(this);
    }
		//根据是否在事务执行，选择线程池
    Executor getQueryExecutor() {
        if (mInTransaction) {
            return mDatabase.getTransactionExecutor();
        } else {
            return mDatabase.getQueryExecutor();
        }
    }
}
```

`InvalidationTracker`是Room处理数据变化通过Observer异步通知数据更新的核心类，`InvalidationTracker`中的 `Observer`是核心类，`Observer`监听数据库的数据变化

```java
public abstract static class Observer {
    final String[] mTables;

    @SuppressWarnings("unused")
    protected Observer(@NonNull String firstTable, String... rest) {
        mTables = Arrays.copyOf(rest, rest.length + 1);
        mTables[rest.length] = firstTable;
    }

    public Observer(@NonNull String[] tables) {
        mTables = Arrays.copyOf(tables, tables.length);
    }

		//数据有变化时，通过此方法更新UI
    public abstract void onInvalidated(@NonNull Set<String> tables);

    boolean isRemote() {
        return false;
    }
}
```

通过使用`WeakObserver`实现我们只需要添加，不需要考虑移除，在GC操作时会自动移除。

通过使用`ObserverWrapper`标记Observer监听的是那个表，在数据有变化时，遍历此集合找出需要发出通知的Observer。

接下来看张流程图，来解释数据变化更新UI的整个过程，主要是在`InvalidationTracker`的几个方法中实现

![image-20220521205125477](http://qike.hilary.top/uPic/image-20220521205125477.png)

`InvalidationTracker`记录者数据库表，监听着表的变化，虽然类很大核心代码也就那几个方法，上图展示了数据变化到通知UI的流程，Room是在我们操作数据语句时，添加到事务中，在事务线束后，把变化的表通过查找记录的观察列表，调用符合条件的监听者，把数据刷新到UI中。

