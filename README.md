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
