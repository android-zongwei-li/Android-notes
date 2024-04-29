Android采用了SQLite数据库来实现数据库持久化数据，但是使用Android原生提供的API来操作SQLite数据库，代码量大，且编写起来容易出错，于是市面上开源了许多框架，如GreenDAO、ORMLite等等。Google在Jetpack中提供了一种新的数据库组件，即Room，让开发者能够更简单高效地进行数据库开发。

Room中有两个比较重要的概念：

- Entity：对应数据库中的一张表，在Java中对应一个实体类，实体类中的属性对应数据库中的字段；
- Dao：数据访问对象，开发者可以通过Dao来进行数据操作。

## Room

### 基本使用

添加依赖：

```java
def room_version = "2.2.6"

implementation "androidx.room:room-runtime:$room_version"
annotationProcessor "androidx.room:room-compiler:$room_version"
```

作为示例代码，我们创建一个名为“person”的数据库表，在表中主要是id、name、grade、isMen和extraInfo等几个字段，首先需要创建对应的Entity实体类：

```java
@Entity(tableName = "person")
public class person {
    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id", typeAffinity = ColumnInfo.INTEGER)
    int mId;

    @ColumnInfo(name = "name", typeAffinity = ColumnInfo.TEXT)
    String name;

    @ColumnInfo(name = "grade", typeAffinity = ColumnInfo.REAL)
    Double grade;

    @ColumnInfo(name = "isMan", typeAffinity = ColumnInfo.INTEGER)
    Boolean isMan;

    @Ignore
    String extraInfo;

    public person(String name, Double grade, Boolean isMan) {
        this.name = name;
        this.grade = grade;
        this.isMan = isMan;
    }

    @Ignore
    public person(String name, Double grade, Boolean isMan, String extraInfo) {
        this.name = name;
        this.grade = grade;
        this.isMan = isMan;
        this.extraInfo = extraInfo;
    }

    @Override
    public String toString() {
        return "person{" +
                "mId=" + mId +
                ", name='" + name + '\'' +
                ", grade=" + grade +
                ", isMan=" + isMan +
                ", extraInfo='" + extraInfo + '\'' +
                '}';
    }
}
```

其中各注解：

- @Entity：修饰类，将该实体类与数据库表对应起来，其中tableName参数值表示对应的表名；
- @PrimaryKey：修饰属性：表示将该字段作为表中的主键，autoGenerate = true表示自动生成;
- @ColumnInfo:修饰属性，将该属性与数据库表中字段对应，name参数表示数据库表中的字段名，typeAffinity表示字段类型；
- @Ignore：修饰方法和属性，用来告诉Room忽略该字段或方法，由于Room只能识别Entity实体类的一个构造方法，所以如果实体类中需要有多个构造方法的时候，其余的需要使用@Ignore修饰。

有了Entity实体类，接下来还需要创建一个Dao工具类：

```java
@Dao
public interface PersonDao {
    String tableName = "person";

    @Insert
    void insertPerson(Person person);

    @Delete
    void deletePerson(Person person);

    @Query("DELETE FROM " + tableName + " WHERE id=:id")
    void deletePerson(int id);

    @Update
    void updatePerson(Person person);

    @Query("SELECT * FROM " + tableName)
    List<Person> queryPersons();

    @Query("SELECT * FROM " + tableName + "  WHERE id = :id")
    Person queryPerson(int id);
}
```

定义的Dao类是一个接口，需要使用@Dao注解修饰，将来项目build之后，Room会生成对应的Impl类。

在Dao工具类中定义出所有的数据库操作，并且使用对应的注解修饰：

- @Insert：插入数据；
- @Delete：删除数据；
- @Update：更新数据；
- @Query：查找数据，此注解需要传入SQL语句作为参数，SQL语句用来限定查询的条件，注意如果是按条件删除数据等操作，也可以使用此注解来修饰。

接下来就是Database类了，此类主要是用来创建数据库：

```java
@Database(entities = {Person.class}, version = 1)
abstract public class PersonDataBase extends RoomDatabase {
    private static final String DATABASE_NAME = "person.db";

    abstract PersonDao mPersonDao();

    private static PersonDataBase mPersonDataBase = null;

    public static PersonDataBase getInstance(Context context) {
        if (mPersonDataBase == null) {
            synchronized (PersonDataBase.class) {
                mPersonDataBase = Room.databaseBuilder(
                        context.getApplicationContext(),
                        PersonDataBase.class,
                        DATABASE_NAME
                ).build();
            }
        }
        return mPersonDataBase;
    }
}
```

如果直接按照上面的写法，会出现以下错误信息：

> Schema export directory is not provided to the annotation processor so we cannot export the schema. You can either provide `room.schemaLocation` annotation processor argument OR set exportSchema to false.

上面的错误信息已经提供了两种解决方法：

- 给 RoomDatabase 设置 exportSchema = false。

```
@Database(entities = {Person.class}, version = 1, exportSchema = false)
abstract public class PersonDataBase extends RoomDatabase {
    abstract PersonDao mPersonDao();
}
```

- 在你的 APP 或者 module 的 `build.gradle` 中添加以下注解信息：

```
android {
    ...
    defaultConfig {
        ...
        //指定room.schemaLocation生成的文件路径
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
```

PersonDatabase是一个抽象类，其中核心有两个方法，第一个就是获取PersonDatabase实例，一般情况下，整个项目中有一个PersonDatabase实例即可，所以我们通常使用单例模式。第二个就是获取其对应的Dao工具类。

创建Database类的时候，需要继承自RoomDatabase类，并使用@Database注解，其中entities参数值是一数组，对应此数据库中的所有表，version表示数据库版本号。

使用Room.databaseBuilder初始化Database对象，需要注意传入的Context是整个应用程序的Context。

接下来就可以使用Room进行数据库增删改查了，由于数据库操作是耗时操作，所以Room强制要求所有的增删改查不能在主线程中执行，这里我们使用Kotlin的协程来实现：

```java
public class RoomActivity extends AppCompatActivity {
    private Context mContext = this;

    @BindView(R.id.btn_insert)
    Button btnInsert;
    @BindView(R.id.btn_delete_by_id)
    Button btnDeleteById;
    @BindView(R.id.btn_delete)
    Button btnDelete;
    @BindView(R.id.btn_update)
    Button btnUpdate;
    @BindView(R.id.btn_query)
    Button btnQuery;
    @BindView(R.id.btn_query_by_id)
    Button btnQueryById;

    private int i = 0;
    private Person p = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_room);
        ButterKnife.bind(this);
    }

    @OnClick({R.id.btn_insert, R.id.btn_delete_by_id, R.id.btn_delete, R.id.btn_update, R.id.btn_query, R.id.btn_query_by_id})
    public void onViewClicked(View view) {
        switch (view.getId()) {
            case R.id.btn_insert:
                insert();
                break;
            case R.id.btn_delete_by_id:
                deleteById();
                break;
            case R.id.btn_delete:
                delete();
                break;
            case R.id.btn_update:
                update();
                break;
            case R.id.btn_query:
                query();
                break;
            case R.id.btn_query_by_id:
                queryById();
                break;
        }
    }

    private void queryById() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Person person = PersonDataBase.getInstance(mContext).mPersonDao().queryPerson(i);
                if (person != null) {
                    Log.e("tag", "queryById: " + person.toString());
                }
            }
        }).start();
    }

    private void query() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                List<Person> persons = PersonDataBase.getInstance(mContext).mPersonDao().queryPersons();
                if (persons != null && !persons.isEmpty()) {
                    for (Person p : persons) {
                        Log.e("tag", "query all: " + p.toString());
                    }
                }
            }
        }).start();
    }

    private void update() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                PersonDataBase.getInstance(mContext).mPersonDao().updatePerson(p);
            }
        }).start();
    }

    /**
     * 没有成功
     */
    private void delete() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                PersonDataBase.getInstance(mContext).mPersonDao().deletePerson(p);
            }
        }).start();
    }

    private void deleteById() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                PersonDataBase.getInstance(mContext).mPersonDao().deletePerson(i);
            }
        }).start();
    }

    private void insert() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                p = new Person(i + "", (double) i, i % 2 == 0);
                i++;
                PersonDataBase.getInstance(mContext).mPersonDao().insertPerson(p);
            }
        }).start();
    }
}
```

activity_room.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".room.RoomActivity">

    <Button
        android:id="@+id/btn_insert"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="insert" />

    <Button
        android:id="@+id/btn_delete_by_id"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="delete_by_id" />

    <Button
        android:id="@+id/btn_delete"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="delete" />

    <Button
        android:id="@+id/btn_update"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="update" />

    <Button
        android:id="@+id/btn_query"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="query" />

    <Button
        android:id="@+id/btn_query_by_id"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="query_by_id" />
</LinearLayout>
```

### Room+ViewModel、LiveData

ViewModel+LiveData的组合，可以使得数据不受UI组件生命周期影响的同时，还能时时监测到数据值的变化，那么如果再结合Room使用，就可以当数据库内容发生变化的时候，主动通知到UI组件，就不需要每次都主动地去执行一些命令。

从上一小节的介绍中得知，创建RoomDatabase实例的时候，需要用到Application的Context，如果使用普通的ViewModel的话，就需要个ViewModel传入Context，由于ViewModel的特性，传入Context会造成内存泄漏，所以我们可以使用ViewModel的子类AndroidViewModel，其正好提供了Application的Context。

在Dao中，定义查询接口的时候，返回值使用LiveData封装一下：

```java
@Query("SELECT * FROM " + tableName)
LiveData<List<Person>> queryPersons();
```

创建ViewModel：

```java
public class RoomViewModel extends AndroidViewModel {
    PersonDataBase mPersonDataBase;
    LiveData<List<Person>> mLiveData;

    public RoomViewModel(@NonNull Application application) {
        super(application);
        mPersonDataBase = PersonDataBase.getInstance(application);
        mLiveData = mPersonDataBase.mPersonDao().queryPersons();
    }
}
```

在UI组件中通过Dao工具获取到LiveData对象，对其进行observe操作即可。

```java
LiveData<List<Person>> listLiveData = PersonDataBase.getInstance(this).mPersonDao().queryPersons();
listLiveData.observe(this, new Observer<List<Person>>() {
    @Override
    public void onChanged(List<Person> people) {
        Log.e("onChanged", people.toString());
    }
});
```

这样操作后，当我们对数据库进行任何数据改动时，都会监测到，时时刷新UI。

### 数据库升级（<u>注：还没有成功</u>）

在需求迭代过程中，可能会对数据库表结构加一些新的字段，遇到这种情况，就需要将数据库版本号进行升级，Room提供了Migration类，用以实现数据库升级。

Migration类的构造方法接收两个参数：startVersion、endVersion，startVersion表示当前数据库版本（旧版本），endVersion为新版本，创建Migration对象：

```java
    private static Migration mMigration1_2 = new Migration(1, 2) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            Log.d("PersonDataBase", "升级");
        }
    };
```

如上述代码，如果从版本1升级到版本2，就在创建Migration对象的时候两个参数分别传入1和2。

在migrate方法中，可以通过database的execSQL()方法，传入对应的SQL语句，即可对数据库进行升级管理，如复制一个完全一样的表等等。

在初始化Database对象的时候，调用addMigratioon()方法设置给Database：

```java
mPersonDataBase = Room.databaseBuilder(
                        context.getApplicationContext(),
                        PersonDataBase.class,
                        DATABASE_NAME
                )
                        .addMigrations(mMigration1_2)
                        .build();
```

如果数据库版本号从1升级到3，系统会优先匹配1-3的Margation，如果没有的话就会依次执行 1-2，2-3。

如果升级数据库但没有匹配到对应的Margation，程序就会奔溃，我们可以在初始化Database对象的时候，加入fallbackToDestructiveMigration()语句：

```java
mPersonDataBase = Room.databaseBuilder(
                        context.getApplicationContext(),
                        PersonDataBase.class,
                        DATABASE_NAME
                )
                        .addMigrations(mMigration1_2)
                        .fallbackToDestructiveMigration()
                        .build();
```

### schema文件

如果想查看数据库内容，Room提供了Schema文件，其中包含了数据库的所有基本信息，我们需要在项目的build.gradle文件中进行如下配置，为其指定文件的导出位置：

```scheme
android {

    defaultConfig {

        javaCompileOptions{
            annotationProcessorOptions{
                arguments=["room.schemaLocation":"$projectDir/ schemas".toString()]
            }
        }
    }
 }
```

最终生成的文件如下：其中包含了数据库版本号，所有的表信息等等。

```scheme
{
  "formatVersion": 1,
  "database": {
    "version": 1,
    "identityHash": "868952efc8c86b0e287ea8aee79327c5",
    "entities": [
      {
        "tableName": "person",
        "createSql": "CREATE TABLE IF NOT EXISTS `${TABLE_NAME}` (`id` INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, `name` TEXT, `grade` REAL, `isMan` INTEGER)",
        "fields": [
          {
            "fieldPath": "mId",
            "columnName": "id",
            "affinity": "INTEGER",
            "notNull": true
          },
          {
            "fieldPath": "name",
            "columnName": "name",
            "affinity": "TEXT",
            "notNull": false
          },
          {
            "fieldPath": "grade",
            "columnName": "grade",
            "affinity": "REAL",
            "notNull": false
          },
          {
            "fieldPath": "isMan",
            "columnName": "isMan",
            "affinity": "INTEGER",
            "notNull": false
          }
        ],
        "primaryKey": {
          "columnNames": [
            "id"
          ],
          "autoGenerate": true
        },
        "indices": [],
        "foreignKeys": []
      }
    ],
    "views": [],
    "setupQueries": [
      "CREATE TABLE IF NOT EXISTS room_master_table (id INTEGER PRIMARY KEY,identity_hash TEXT)",
      "INSERT OR REPLACE INTO room_master_table (id,identity_hash) VALUES(42, '868952efc8c86b0e287ea8aee79327c5')"
    ]
  }
}
```

