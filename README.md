# 期中考试实验说明文档
该实验是在原有的Note Pad上添加两个功能：
1. NoteList中显示条目增加时间戳显示
2. 添加笔记查询功能（根据标题查询）
接下来对以上需要添加的两个功能进行说明与演示
## 添加时间戳
### 需求分析
1. 只需在notepadprovider类中修改insert方法，用时间戳来格式化时间格式，并添加到数据库中即可，关键代码如下：
```
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
        }
```
2. 在进行笔记更新的时候，只需修改NoteEditor类中的updateNote方法，用时间戳来格式化时间格式，并添加到数据库中即可，关键代码如下：
```        ContentValues values = new ContentValues();
        //将格式化后的时间戳数据更新到数据库中
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
```
### 运行结果
![增加时间戳1](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E5%A2%9E%E5%8A%A0%E6%97%B6%E9%97%B4%E6%88%B31.png)<br />
![增加时间戳2](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E5%A2%9E%E5%8A%A0%E6%97%B6%E9%97%B4%E6%88%B32.png)
## 添加笔记查询功能
### 需求分析
1. 只需在lists_optioin_menu.xml菜单文件中添加搜索菜单即可，关键代码如下：
```
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```
2. 新建布局文件note_search.xml，该文件主要用于放置搜索框searchview和使用Listview放置搜索结果，代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
3. 新建Activity，名为NoteSearch,绑定布局文件note_search.xml，用于执行笔记搜索操作，代码如下：
将搜索结果放在cursor中，用SimpleCursorAdaptor进行封装
```
public boolean onQueryTextChange(String newText) {
        //搜索语句，搜索结果存放在cursor中，使用SimpleCursorAdapter进行数据的装填
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
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
```
使用原有NoteList文件中的onListItemClick方法，实现点击搜索结果，跳转至Note编辑页面
```
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
```
### 运行结果
![搜索功能1](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD1.png)<br />
![搜索功能2](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD2.png)<br />
![搜索功能3](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD3.png)<br />
![搜索功能4](https://github.com/ysw990312/AndroidPrograms/blob/master/NotePad-master/Pictures/%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD4.png)
