# 内容提供器









## 代码范例

### 访问提供程序

```java
// Queries the user dictionary and returns results
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause                    // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows
```

### URI 末位追加 ID

> - `Uri` 和 `Uri.Builder` 类包含根据字符串构建格式规范的 URI 对象的便利方法。 `ContentUris` 包含一些可以将 ID 值轻松追加到 URI 后的方法。

```java
Uri singleUri = ContentUris.withAppendedId(UserDictionary.Words.CONTENT_URI,4);
```

### 构建查询

> - 定义某些用于访问用户字典提供程序的变量

```java

// A "projection" defines the columns that will be returned for each row
String[] mProjection =
{
    UserDictionary.Words._ID,    // Contract class constant for the _ID column name
    UserDictionary.Words.WORD,   // Contract class constant for the word column name
    UserDictionary.Words.LOCALE  // Contract class constant for the locale column name
};

// Defines a string to contain the selection clause
String mSelectionClause = null;

// Initializes an array to contain selection arguments
String[] mSelectionArgs = {""};

```

> - 使用 `ContentResolver.query()`
> - 查询应该返回的列集被称为**投影**（变量 `mProjection`）

```java
/*
 * This defines a one-element String array to contain the selection argument.
 */
String[] mSelectionArgs = {""};

// Gets a word from the UI
mSearchString = mSearchWord.getText().toString();

// Remember to insert code here to check for invalid or malicious input.

// If the word is the empty string, gets everything
if (TextUtils.isEmpty(mSearchString)) {
    // Setting the selection clause to null will return all words
    mSelectionClause = null;
    mSelectionArgs[0] = "";

} else {
    // Constructs a selection clause that matches the word that the user entered.
    mSelectionClause = UserDictionary.Words.WORD + " = ?";

    // Moves the user's input string to the selection arguments.
    mSelectionArgs[0] = mSearchString;

}

// Does a query against the table and returns a Cursor object
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,  // The content URI of the words table
    mProjection,                       // The columns to return for each row
    mSelectionClause                   // Either null, or the word the user entered
    mSelectionArgs,                    // Either empty, or the string the user entered
    mSortOrder);                       // The sort order for the returned rows

// Some providers return null if an error occurs, others throw an exception
if (null == mCursor) {
    /*
     * Insert code here to handle the error. Be sure not to use the cursor! You may want to
     * call android.util.Log.e() to log this error.
     *
     */
// If the Cursor is empty, the provider found no matches
} else if (mCursor.getCount() < 1) {

    /*
     * Insert code here to notify the user that the search was unsuccessful. This isn't necessarily
     * an error. You may want to offer the user the option to insert a new row, or re-type the
     * search term.
     */

} else {
    // Insert code here to do something with the results

}
```

> - 防止恶意输入

```java
//part 1 :危险
// Constructs a selection clause by concatenating the user's input to the column name
String mSelectionClause =  "var = " + mUserInput;

//part 2:安全 （用户输入直接受查询约束，而不解释为 SQL 语句的一部分）
// Constructs a selection clause with a replaceable parameter
String mSelectionClause =  "var = ?";
```

### 显示查询结果

> - `ContentResolver.query()` 客户端方法始终会返回符合以下条件的 `Cursor`：包含查询的投影为匹配查询选择条件的行指定的列。 `Cursor` 对象为其包含的行和列提供随机读取访问权限。 通过使用 `Cursor` 方法，您可以循环访问结果中的行、确定每个列的数据类型、从列中获取数据，并检查结果的其他属性。 某些 `Cursor` 实现会在提供程序的数据发生更改时自动更新对象和/或在 `Cursor` 更改时触发观察程序对象中的方法。
> - 如果没有与选择条件匹配的行，则提供程序会返回 `Cursor.getCount()` 为 0（空游标）的 `Cursor` 对象。
> - 如果出现内部错误，查询结果将取决于具体的提供程序。它可能会选择返回 `null`，或引发 `Exception`。
> - 由于 `Cursor` 是行“列表”，因此显示 `Cursor` 内容的良好方式是通过 `SimpleCursorAdapter` 将其与 `ListView` 关联。

```java
// Defines a list of columns to retrieve from the Cursor and load into an output row
String[] mWordListColumns =
{
    UserDictionary.Words.WORD,   // Contract class constant containing the word column name
    UserDictionary.Words.LOCALE  // Contract class constant containing the locale column name
};

// Defines a list of View IDs that will receive the Cursor columns for each row
int[] mWordListItems = { R.id.dictWord, R.id.locale};

// Creates a new SimpleCursorAdapter
mCursorAdapter = new SimpleCursorAdapter(
    getApplicationContext(),               // The application's Context object
    R.layout.wordlistrow,                  // A layout in XML for one row in the ListView
    mCursor,                               // The result from the query
    mWordListColumns,                      // A string array of column names in the cursor
    mWordListItems,                        // An integer array of view IDs in the row layout
    0);                                    // Flags (usually none are needed)

// Sets the adapter for the ListView
mWordList.setAdapter(mCursorAdapter);
```

### 从查询结果中获取数据

```java

// Determine the column index of the column named "word"
int index = mCursor.getColumnIndex(UserDictionary.Words.WORD);

/*
 * Only executes if the cursor is valid. The User Dictionary Provider returns null if
 * an internal error occurs. Other providers may throw an Exception instead of returning null.
 */

if (mCursor != null) {
    /*
     * Moves to the next row in the cursor. Before the first movement in the cursor, the
     * "row pointer" is -1, and if you try to retrieve data at that position you will get an
     * exception.
     */
    while (mCursor.moveToNext()) {

        // Gets the value from the column.
        newWord = mCursor.getString(index);

        // Insert code here to process the retrieved word.

        ...

        // end of while loop
    }
} else {

    // Insert code here to report an error if the cursor is null or the provider threw an exception.
}
```

### 插入数据

> - 新行的数据会进入单个 `ContentValues` 对象中，该对象在形式上与单行游标类似。 此对象中的列不需要具有相同的数据类型，如果您不想指定值，则可以使用 `ContentValues.putNull()` 将列设置为 `null`

```java
// Defines a new Uri object that receives the result of the insertion
Uri mNewUri;

...

// Defines an object to contain the new values to insert
ContentValues mNewValues = new ContentValues();

/*
 * Sets the values of each column and inserts the word. The arguments to the "put"
 * method are "column name" and "value"
 */
mNewValues.put(UserDictionary.Words.APP_ID, "example.user");
mNewValues.put(UserDictionary.Words.LOCALE, "en_US");
mNewValues.put(UserDictionary.Words.WORD, "insert");
mNewValues.put(UserDictionary.Words.FREQUENCY, "100");

mNewUri = getContentResolver().insert(
    UserDictionary.Word.CONTENT_URI,   // the user dictionary content URI
    mNewValues                          // the values to insert
);
```

> - `newUri` 中返回的内容 URI 会按照以下格式标识新添加的行

```java
content://user_dictionary/words/<id_value>
```

> - 要从返回的 `Uri` 中获取 `_ID` 的值，请调用 `ContentUris.parseId()`

### 更新数据

> - 返回值是已更新的行数

```java
// Defines an object to contain the updated values
ContentValues mUpdateValues = new ContentValues();

// Defines selection criteria for the rows you want to update
String mSelectionClause = UserDictionary.Words.LOCALE +  "LIKE ?";
String[] mSelectionArgs = {"en_%"};

// Defines a variable to contain the number of updated rows
int mRowsUpdated = 0;

...

/*
 * Sets the updated value and updates the selected words.
 */
mUpdateValues.putNull(UserDictionary.Words.LOCALE);

mRowsUpdated = getContentResolver().update(
    UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI
    mUpdateValues                       // the columns to update
    mSelectionClause                    // the column to select on
    mSelectionArgs                      // the value to compare to
);
```

### 删除数据

> - 返回已删除的行数

```java

// Defines selection criteria for the rows you want to delete
String mSelectionClause = UserDictionary.Words.APP_ID + " LIKE ?";
String[] mSelectionArgs = {"user"};

// Defines a variable to contain the number of rows deleted
int mRowsDeleted = 0;

...

// Deletes the words that match the selection criteria
mRowsDeleted = getContentResolver().delete(
    UserDictionary.Words.CONTENT_URI,   // the user dictionary content URI
    mSelectionClause                    // the column to select on
    mSelectionArgs                      // the value to compare to
);
```



