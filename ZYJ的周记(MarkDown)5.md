### 5月11号: （主题: 通讯录）

#### 功能：
- 从本地系统导入联系人
- 信息包括：多个电话（有短信、拨号按键），名称
- 可以对联系人进行分组
- 所以数据都能增删查改

- 信息包括：生日（通知栏生日提醒）
- 联系人有头像
- 可以搜索联系人（模糊搜索）
- 有索引栏
- 登陆注册，将联系人上传至云端

##### 1.从本地系统导入联系人

首先添加权限：

```
<uses-permission android:name="android.permission.READ_CONTACTS" />
```
然后运行时申请权限：

```
if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_CONTACTS}, 1);
}
```
获取联系人的姓名和电话号码：
```
Cursor cursor = activity.getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
if (cursor != null) {
    while (cursor.moveToNext()) {
        String displayName = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
        String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
        ...
        ...
    }
cursor.close();
```
- **ContactsContract.CommonDataKinds.Phone.CONTENT_URI**是已经Uri.parse()之后得到的结果。
- **ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME**是联系人的名字。
- **ContactsContract.CommonDataKinds.Phone.NUMBER**是联系人的电话号码。

在这里通过Activity的getContentResolver方法获得一个ContentResolver对象，再使用这个对象的query方法得到Cursor，然后就可以对联系人的资料进行遍历了。


##### 2.短信、拨号界面

这里直接使用Intent来进入系统的拨号、发短信的界面：

```
final String call = "tel:" + mNumberList.get(position);
final String message = "smsto:" + mNumberList.get(position);
        
        
Intent intent = new Intent(Intent.ACTION_DIAL);
intent.setData(Uri.parse(call));
mContext.startActivity(intent);

Intent intent = new Intent(Intent.ACTION_SENDTO);
intent.setData(Uri.parse(message));
mContext.startActivity(intent);
```
在Intent的构造方法中，填入了一个Intent自带的常量，具体的可以看源码，会有一些直接发送、拨号的常量，我这里的常量只是用来跳转到发送和拨号界面就好了。

##### 3.联系人分组：
在原有的基础上，新增一个群组数据库。这是我头一次在一个项目中建两个库。

```
private final static String CREATE_SCHEDULE = "create table Groups(" +
    "name text primary key," +
    "member text)";
```
内容很简单，只有群组名和群组成员。
思路：
- 每次保存群组的时候，将所有的成员名字拼成一个字符串，每个群员之间用 "," 逗号隔开。
- 在读取群组成员的时候，用数组的split(",")方法拆分数组
```
String[] strings = groupMember.split(",");
```
这样就行了，思路来源于我之前做的日历APP中添加多张图片的逻辑。

![image](https://note.youdao.com/favicon.ico)

做的时候遇到问题：在制作群组的时候,使用了checkbox多选框来选择联系人，然后在Adapter中，想通过
```
checkBox.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        
    }
});
```
来监听CheckBox的选择情况，但不幸的是，CheckBox的所有逻辑代码都在Adapter的onBindViewHolder方法中，也就是每次Item滚动到屏幕中时，都会调用这个方法。

修改群组时，已经在群组中的人要被选中，也就是CheckBox要setChecked(true)，然而这一举动会被监听到，进而触发不必要的事件。

于是灵活变通，做了以下修改：

```
checkBox.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        ...
        if (checkBox.isChecked()) {
            ...
        } else {
            ...
        }
    }
});
```
CheckBox的点击不会被checked消费掉，所以还能获得onClick这一动作。

##### 4.数据都能增删查改
也不知道频繁开关读写数据库是不是不太合理，不过我的手机在进行跟数据库有关的操作时完全不会卡，可能是sqlite性能优越吧。如果有必要的话，下个APP可以改一改，**在onCreate时统一查找**，数据作为**数组**在整个APP运行时存在，**对数组**进行**修改、插入、删除**，在onStop时统一把**数组解析**，然后对数据库**插入、升级**。不过删除的话好像得再想想。

##### 5.生日（通知栏生日提醒）

Notification通知&&Alarm机制
*==//TODO==*

##### 6.联系人头像
图片、拍照功能在MyCalendar中已经实现，这里照搬。

##### 7以搜索联系人（模糊搜索）
- 使用SearchView：

```
<item
    android:id="@+id/menu_search"
    android:orderInCategory="100"
    android:title="搜索联系人"
    app:actionViewClass="android.support.v7.widget.SearchView"
    app:showAsAction="always" />
```
actionViewClass指定v7库提供的SearchView，非常美观

![image](https://note.youdao.com/favicon.ico)


```
private void setSearchView(final Menu menu) {
//设置SearchView
    MenuItem menuItem = menu.findItem(R.id.menu_search);
    mSearchView = (SearchView) MenuItemCompat.getActionView(menuItem);
    mSearchView.setQueryHint("查找联系人..");
    mSearchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
        @Override
        public boolean onQueryTextSubmit(String query) {
            mRecyclerView.setAdapter(new ContactAdapter(DataToArrayList.getListFromDatabase(query)));
            return false;
        }

        @Override
        public boolean onQueryTextChange(String newText) {
            mRecyclerView.setAdapter(new ContactAdapter(DataToArrayList.getListFromDatabase(newText)));
            return false;
        }
    });
}
```
开头两句是很重要的，要通过MenuItemCompat的getActionView方法来获得SearchView，这个方法中传入的参数是Menu的实例。

试过findViewById来找SearchView直接崩了。

这里设了一个监听，有两个重写的方法
1. onQueryTextSubmit在按搜索按钮的时候会被调用
2. onQueryTextChange在搜索框里面的字改变的时候就会被调用

其实这个SearchView挺简洁大方的，还不错吧。

再说说怎么实现模糊搜索：

```
Cursor cursor = db.query("Contact", null, "spell LIKE ? ", new String[]{"%" + where + "%"}, null, null, null);
```
网上提到不止一种写法，这里使用了其中一种："xxx LIKE ?" ,new String[]{"%" + where + "%"}，这种数据库自带的模糊搜索非常强，一个text无论是开头还是中间抑或是结尾，只要有这个匹配的就会搜索成功。

- 我还自创了一种独门方法，可以实现非常强大的搜索功能：

将每个人的姓名、电话号码、生日、公司、职位、拼音小写、拼音大写、名字拼音缩写全部缩成一个字符串，然后存入数据库中，那么模糊搜索就能实现无论是搜这个人的什么资料，都能把这个人找出来。

![image](https://note.youdao.com/favicon.ico)

##### 8.索引栏

- 实现右侧字母索引栏

自定义View的制作真的令我受益匪浅。


```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    int height = getHeight();
    int width = getWidth();
    int singleHeight = height / 26;
    Paint paint = new Paint();
    stringlist = ALPHABET.split(" ");
    for (int i = 0; i < stringlist.length; i++) {
        paint.setColor(Color.parseColor("#666666"));
        paint.setTextSize(35);
        paint.setTypeface(Typeface.DEFAULT);
        paint.setAntiAlias(true);
        float xPos = (width - paint.measureText(stringlist[i])) / 2;
        float yPos = singleHeight * i + singleHeight;
        canvas.drawText(stringlist[i], xPos, yPos, paint);
        paint.reset();
    }
}
    
OnLetterTouchedChangeListener onLetterTouchedChangeListener = null;

public void setOnLetterTouchedChangeListener(OnLetterTouchedChangeListener listener) {
    this.onLetterTouchedChangeListener = listener;
}

public interface OnLetterTouchedChangeListener {
    void onTouchLetterChange(String letter);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
        case MotionEvent.ACTION_MOVE:
        case MotionEvent.ACTION_UP:
        case MotionEvent.ACTION_CANCEL:
            float touchYpos = event.getY();
            int currentTouchIndex = (int) (touchYpos / getHeight() * 26);
            if (onLetterTouchedChangeListener != null && currentTouchIndex < 26 && currentTouchIndex >= 0) {
                onLetterTouchedChangeListener.onTouchLetterChange(stringlist[currentTouchIndex]);
            }
    }
    return true;
}
```
1. 重写了onDraw方法，通过getWidth和getHeight来获得当前View的长和宽。
2. 长/26得到每个字母的高度
3. 获得画笔Paint的实例
4. 画笔设置了
    1. 字体颜色
    2. 字体大小
    3. 字体形式（默认）
    4. 抗锯齿
5. 计算出字母应该在的xy坐标
6. canvas使用drawText把字母画出来
7. 重置画笔


8. 声明接口OnLetterTouchedChangeListener
9. 提供外界的方法setOnLetterTouchedChangeListener
10. View设置onTouch监听，给OnLetterTouchedChangeListener提供当前触碰到的字母

##### 9.登陆注册，将联系人上传至云端
*==//TODO==*