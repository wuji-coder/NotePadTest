# 一、实验名称
NOTEPAD笔记本应用
# 二、实现功能截图
#### 1.界面菜单列表
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/1.png)
#### 2.笔记编辑界面菜单列表
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/2.png)
#### 3.新建笔记以及显示创建时间
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/3.png)
#### 4.按标题搜索笔记
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/4.png)
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/5.png)
#### 5.编辑笔记标题
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/6.png)
#### 6.编辑笔记后显示修改时间
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/7.png)
#### 7.删除笔记
![image](https://github.com/wuji-coder/NotePadTest/blob/master/image/8.png)
# 三、相关代码及分析
#### 1.NoteList中显示条目增加时间戳显示

先在noteslist_item.xml中增加一个用来显示时间的TextView
```java
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingStart="20sp"
        android:paddingEnd="20sp"
        android:paddingLeft="20sp"
        android:textColor="@color/white"/>
 ```
然后在NotesList的数据定义中增加修改时间
```java
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展显示时间
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
 ```
接着在NotePadProvider中修改创建时间和修改时间的类型为TEXT
```java
@Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                    + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                    + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " TEXT"
                    + ");");
        }
```
在NotesList中使用SimpleCursorAdapter来装配数据，首先查询数据库的内容，使用ContentProvider默认的URI
```java
Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
```
装配的时候需要装配相应的日期，修改dataColumns,viewIDs这两个参数
```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1 ,R.id.text1_time};
```
通过SimpleCursorAdapter来进行装配
```java
SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
```
在NoteList创建修改日期格式的方法,用于将长整型的数据转换为时间数据
```java
public static String getFormatedDateTime(String pattern,long dateTime){
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat(pattern);
        return simpleDateFormat.format(new Date(dateTime+1000*60*60*12));
    }
```
在NotePadProvider中的insert方法和NoteEditor中的updateNote方法里修改创建数据时和修改数据时的时间格式
```java
String now = getFormatedDateTime("yyyy-MM-dd HH:mm:ss",System.currentTimeMillis());
```
```java
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, getFormatedDateTime("yyyy-MM-dd HH:mm:ss",System.currentTimeMillis()));
```
#### 2.添加笔记查询功能（根据标题查询）

先在list_options_menu.xml中新建一个查询的按钮
```java
<item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        app:showAsAction="ifRoom|withText">
</item>
```
然后在NotesList中的onOptionsItemSelected的switch中添加对应menu_search的case，增加点击之后的反应
```java
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
在layout中新建布局文件note_search_list.xml，先布局搜索页面
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:LinearLayout="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        LinearLayout:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
新建一个名为NoteSearch的类，由于搜索出来的也是笔记列表，所以可以模仿NoteList的activity继承ListActivity。搜索采用SearchView实现，并对SearchView文本变化设置监听，然后通过实现implements SearchView.OnQueryTextListener接口来完成查询
```java
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展显示时间
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+newText+"%" };//查询条件参数，配合selection参数使用,%通配多个字符
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择
                selectionArgs,                    // 查询条件参数，配合selection参数使用
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
最后在AndroidManifest.xml注册NoteSearch
```java
        <activity
            android:name="NoteSearch"
            android:label="@string/title_notes_search">
        </activity>
```
