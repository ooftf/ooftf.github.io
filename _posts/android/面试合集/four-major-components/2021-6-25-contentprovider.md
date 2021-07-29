---
title: ContentProvider
---

* ContentProvider是Android中提供的专门用于不同应用间进行数据共享的方式
* ContentProvider的底层实现同样也是Binder
* 六个需要重写的方法 ： onCreate、query、update、insert、delete和getType
* 除了onCreate由系统回调并运行在主线程里，其他五个方法均由外界回调并运行在Binder线程池中
* getType用来返回一个Uri请求所对应的MIME类型（媒体类型），比如图片、视频等，如果我们的应用不关注这个选项，可以直接在这个方法中返回null或者“*/*
* 虽然ContentProvider的方法看起来很像数据库操作，但是ContentProvider对底层的数据存储方式没有任何要求，我们既可以使用SQLite数据库，也可以使用普通的文件，甚至可以采用内存中的一个对象来进行数据的存储
* android:authorities 是 Content-Provider 的唯一标识，通过这个属性外部应用就可以访问我们的 ContentProvider，因此，android:authorities必须是唯一的，这里建议在命名的时候加上包名前缀。
* query、update、insert、delete四大方法是存在多线程并发访问的，因此方法内部要做好线程同步。

## ContentProvider 的启动流程

当一个应用启动时，入口方法为ActivityThread的main方法，在main方法中会创建ActivityThread的实例并创建主线程的消息队列，然后在ActivityThread的attach方法中会远程调用AMS的attachApplication方法并将ApplicationThread对象提供给AMS。ApplicationThread是一个Binder对象，它的Binder接口是IApplicationThread，它主要用于ActivityThread和AMS之间的通信。在AMS的attachApplication方法中，会调用ApplicationThread的bindApplication方法，这个过程同样是跨进程完成的，bindApplication的逻辑会经过ActivityThread中的mH Handler切换到ActivityThread中去执行，具体的方法是handleBindApplication。在handleBindApplication方法中，ActivityThread会创建Application对象并加载ContentProvider。需要注意的是，ActivityThread会先加载ContentProvider，然后再调用Application的onCreate方法

![ContentProvider 的启动流程](https://raw.githubusercontent.com/ooftf/Material/master/img/blog/20210729224931.png)
## ContentProvider 示例
```java
public class BookProvider extends ContentProvider {

    private static final String TAG = "BookProvider";

    public static final String AUTHORITY = "com.ryg.chapter_2.book.provider";

    public static final Uri BOOK_CONTENT_URI = Uri.parse("content://"
            + AUTHORITY + "/book");
    public static final Uri USER_CONTENT_URI = Uri.parse("content://"
            + AUTHORITY + "/user");
    public static final int BOOK_URI_CODE = 0;
    public static final int USER_URI_CODE = 1;
    private static final UriMatcher sUriMatcher = new UriMatcher(
            UriMatcher.NO_MATCH);

    static {
        sUriMatcher.addURI(AUTHORITY, "book", BOOK_URI_CODE);
        sUriMatcher.addURI(AUTHORITY, "user", USER_URI_CODE);
    }

    private Context mContext;
    private SQLiteDatabase mDb;

    @Override
    public boolean onCreate() {
        Log.d(TAG, "onCreate, current thread:"
                + Thread.currentThread().getName());
        mContext = getContext();
        //ContentProvider创建时，初始化数据库。注意：这里仅仅是为了演示，实际使用中不推荐在主线程中进行耗时的数据库操作
        initProviderData();
        return true;
    }

    private void initProviderData() {
        mDb = new DbOpenHelper(mContext).getWritableDatabase();
        mDb.execSQL("delete from " + DbOpenHelper.BOOK_TABLE_NAME);
        mDb.execSQL("delete from " + DbOpenHelper.USER_TALBE_NAME);
        mDb.execSQL("insert into book values(3, 'Android'); ");
        mDb.execSQL("insert into book values(4, 'Ios'); ");
        mDb.execSQL("insert into book values(5, 'Html5'); ");
        mDb.execSQL("insert into user values(1, 'jake',1); ");
        mDb.execSQL("insert into user values(2, 'jasmine',0); ");
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        Log.d(TAG, "query, current thread:" + Thread.currentThread().
                getName());
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI: " + uri);
        }
        return mDb.query(table, projection, selection, selectionArgs, null,
                null, sortOrder, null);
    }

    @Override
    public String getType(Uri uri) {
        Log.d(TAG, "getType");
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        Log.d(TAG, "insert");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI: " + uri);
        }
        mDb.insert(table, null, values);
        mContext.getContentResolver().notifyChange(uri, null);
        return uri;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        Log.d(TAG, "delete");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI: " + uri);
        }
        int count = mDb.delete(table, selection, selectionArgs);
        if (count > 0) {
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return count;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        Log.d(TAG, "update");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI: " + uri);
        }
        int row = mDb.update(table, values, selection, selectionArgs);
        if (row > 0) {
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return row;
    }

    private String getTableName(Uri uri) {
        String tableName = null;
        switch (sUriMatcher.match(uri)) {
            case BOOK_URI_CODE:
                tableName = DbOpenHelper.BOOK_TABLE_NAME;
                break;
            case USER_URI_CODE:
                tableName = DbOpenHelper.USER_TALBE_NAME;
                break;
            default:
                break;
        }

        return tableName;
    }
}
            
```
## 使用 ContentResolver 访问数据
```java
public class ProviderActivity extends Activity {
    private static final String TAG = "ProviderActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Uri bookUri = Uri.parse("content://com.ryg.chapter_2.book.provider/
                book");
                ContentValues values = new ContentValues();
        values.put("_id", 6);
        values.put("name", "程序设计的艺术");
        getContentResolver().insert(bookUri, values);
        Cursor bookCursor = getContentResolver().query(bookUri, new String[]
                {"_id", "name"}, null, null, null);
        while (bookCursor.moveToNext()) {
            Book book = new Book();
            book.bookId = bookCursor.getInt(0);
            book.bookName = bookCursor.getString(1);
            Log.d(TAG, "query book:" + book.toString());
        }
        bookCursor.close();

        Uri userUri = Uri.parse("content://com.ryg.chapter_2.book.provider/
                user");
                Cursor userCursor = getContentResolver().query(userUri, new String[]
                        {"_id", "name", "sex"}, null, null, null);
        while (userCursor.moveToNext()) {
            User user = new User();
            user.userId = userCursor.getInt(0);
            user.userName = userCursor.getString(1);
            user.isMale = userCursor.getInt(2) == 1;
            Log.d(TAG, "query user:" + user.toString());
        }
        userCursor.close();
    }
}
```


## ContentObserver
```kotlin
contentResolver.registerContentObserver(
    Uri.parse("content://com.ryg.chapter_2.book.provider/user"),
    false,
    object : ContentObserver(
        Handler(Looper.myLooper())
    ) {
        override fun onChange(selfChange: Boolean) {
            // do something
        }
    })
```



