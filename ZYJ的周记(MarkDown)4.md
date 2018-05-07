### 5月1号: （主题: Material Design）

#### 1. Toolbar

- Toolbar代替ActionBar

在做MyCalendar的时候，为了增大界面的面积，我在style中将AppTheme中的parent改成了Theme.AppCompat.Ligth.NoActionBar

```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>
```
以此达到消去ActionBar的效果。现在，appcompat-7库提供了一个Toolbar控件，结合其他控件时，Toolbar可以实现非常丰富的效果。
那么尝试一下。这次不对MyCalendar动手了，新建一个工程。在主布局中加入以下代码：

```
<android.support.v7.widget.Toolbar
    android:id="@+id/toolBar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="@color/colorPrimary"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light">
</android.support.v7.widget.Toolbar>
```
注意不要选错了，是要v7包里面的Toolbar。这里指定了Toolbar的主题和弹出菜单的主题。
在主活动中，利用findViewById()得到Toolbar的实例，然后用setSupportActionBar()方法将Toolbar的实例传入。

**注意导入的包是**
```
import android.support.v7.app.AppCompatActivity;
```
---

**使用的Toolbar应该是**

```
android.support.v7.widget.Toolbar toolbar = findViewById(R.id.toolBar);
```
然后为这个Toolbar添加一些按钮。在res目录中创建menu文件夹，然后新建一个Menu resource file

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_message"
        android:icon="@drawable/message"
        android:title="@string/message"        
        app:showAsAction="always">
    </item>
    <item
        android:id="@+id/menu_collection"
        android:icon="@drawable/collection"
        android:title="@string/collection"
        app:showAsAction="ifRoom">
    </item>
    <item
        android:id="@+id/menu_empty"
        android:icon="@drawable/empty"
        android:title="@string/empty"
        app:showAsAction="never">

    </item>
</menu>
```
这里加入了三个item，其中
1. icon设置的是图标
2. title设置的显示在菜单时显示的文字
3. showAsAction设置的显示方式
    1. ifRoom 当空间充足时显示
    2. always 永远显示在Toolbar中
    3. nevr 永远不显示在Toolbar中，只显示在菜单里面

然后**重写onCreateOptionsMenu() 方法**

```
@Override
public boolean onCreateOptionsMenu(Menu menu) {   
    getMenuInflater().inflate(R.menu.toolbar_menu, menu);
    return true;
}
```
加载这个menu
可以**重写onOptionItemSelected()方法**来处理各个按钮的点击事件。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/Toolbar1.png)

#### 2.DrawerLayout
- 滑动菜单

DrawerLayout要求有两个子控件，第一个子控件会显示在主屏幕中，第二个控件会作为菜单滑出。

注意第二个控件必须加上**layout_gravity**属性，而不是**garvity**属性。这个属性只有LinearLayout的子控件才有，所以编译器没有提示这个属性，不过我们是一定要加上的，不然菜单无法滑出。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/DrawerLayout.png)

现在在Toolbar的左上角添加一个导航按钮，按下后会打开DrawerLayout。


```
mDrawerLayout = findViewById(R.id.drawer_layout);
ActionBar actionBar = getSupportActionBar();
if (actionBar != null){
    actionBar.setDisplayHomeAsUpEnabled(true);
    actionBar.setHomeAsUpIndicator(R.drawable.other);
}
```
这里将Actionbar自带的一个返回键修改为显示状态，还修改了图标。

---

```
mDrawerLayout.openDrawer(GravityCompat.START);
```
然后用此方法打开DrawerLayout

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/DrawerLayout1.png)

真难看，而且还没有相关的可以调整图标大小的方法，还得自己去ps调图标的大小。

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/DrawerLayout2.png)

就这样吧

#### 3.NavigationView

- 使用NavigationView做出好看的滑动菜单页面

DrawerLayout滑出来的菜单比较单调，不过DesignSupport库中提供了一个控件：NavigationView，帮助我们完成美观的滑动菜单页面

##### 1. 首先添加依赖库

```
implementation 'com.android.support:design:27.1.1'
implementation 'de.hdodenhof:circleimageview:2.1.0'
```
其中第二个是将图片变为圆形的开源库

---

##### 2. 制作NavigationView的menu菜单

跟其他menu的做法一样，略过

--- 
##### 3. 制作NavigationView的head布局

这个也比较随意

---

##### 4.将NavigationView放入DrawerLayout中

```
<android.support.design.widget.NavigationView
    android:id="@+id/nav_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:headerLayout="@layout/nav_header"
    app:menu="@menu/nav_menu">
</android.support.design.widget.NavigationView>
```
比较注意的还是**layout_gravity**属性，**headerLayout**和**menu**就用上面已经做好的menu和head布局

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/NavigationView.png)

看具体需要来调整吧

#### 4.FloatingActionButton

- 悬浮按钮！

就是普通的按钮，多了一种悬浮的画面效果，但是这个按钮的背景取决于colorAccent属性，因此改变background也没有用，因此自己写了一个style:

```
<item name="colorAccent">@color/colorPrimary</item>
```
FloatingActionButton中设置主题：
```
android:theme="@style/MyTheme"
```

FloatingActionButton最重要的就是悬浮的阴影了，没有这个阴影其实和普通的按钮一模一样：

```
android:elevation="8dp"
```
这个值越大，按钮的“高度”越大，阴影就越浅。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/FloatActionButton.png)

其实阴影看起来也没想象这么明显

#### 5.Snackbar
- 增强版的Toast

DesignSupport库提供了一种更加先进的提示工具———Snackbar，Snackbar与Toast不同的地方在于，Snackbar允许在提示中**加入一个可交互的按钮**，而Toast只能被动接受信息，这对于用户体验来说是个不错的选择。


```
Snackbar.make(toolbar, "Hello", Snackbar.LENGTH_LONG).setAction("Undo", new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.d("TAG", "onClick: ");
    }
}).show();
```
make()中，第一个参数是View，只要传入当前布局的任意一个View即可，Snackbar会使用这个View来自动查找最外层的布局。

然后在setAction()方法中设置了一个按钮，第一个参数是按钮显示的文字，第二个参数是一个点击的监听器。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/Snackbar.png)

这个Snackbar把按钮挡住了，为了增强用户体验，那么就要解决这个问题，使用CoordinatorLayout就能轻松解决。

#### 6.CoordinatorLayout

- 增强版的FrameLayout

CoordinatorLayout最出色的地方在于它可以配合其他MaterialDesign控件，自动做出最合理的响应。

要使用CoordinatorLayout，直接在代码中替换掉FrameLayout就好了。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/CoordinatorLayout.png)

#### 7.CardView

- 实现卡片式布局效果

首先添加依赖库

```
implementation 'com.android.support:cardview-v7:27.1.1'
implementation 'com.github.bumptech.glide:glide:3.7.0'
```
第一个是必须要添加的，而第二个是一个开源库，可以十分方便的加载本地图片、网络图片、GIF图片、甚至是本地视频。

---

CardView有什么特别的地方呢？其实CardView可以展示出一个卡片形状的布局，拥有四个圆角，还有FloatActionButton的阴影效果。

看看bilibili客户端的样式：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/CardView2.png)


是不是很明显用了CardView，而且这一个一个的CardView是作为列表的item项显示的，很明显就是用了RecyclerView的网格形式。

那么我现在也用RecyclerView来展示CardView的功能。

1. 新建item布局

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/food_image"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />

        <TextView
            android:id="@+id/food_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp" />

    </LinearLayout>

</android.support.v7.widget.CardView>
```
在一个CardView布局中，加入了一个LinearLayout来线性地摆放了一个ImageView和TextView。**cardCornerRadius**属性指定了圆角的弧度。这样item的布局就做好了。

然后就是加入RecyclerView和自定义Adapter了，这里就没什么好说的了。

选取了一些美食图片，来看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/CardView.png)

效果还行，不过这个列表把Toolbar给挡住了，这也很正常，毕竟这是一个FrameLayout，如果要解决这个问题就需要借助**AppBarLayout**了。

#### 8.AppBarLayout

- 实现Toolbar随着滚动事件而显示/消失

Design Support库中提供了一个工具AppBarLayout，来看看它是怎么用的：


```
<android.support.design.widget.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <android.support.v7.widget.Toolbar
    
        android:id="@+id/toolBar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/ToolbarTheme"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light">

    </android.support.v7.widget.Toolbar>

</android.support.design.widget.AppBarLayout>
        
<android.support.v7.widget.RecyclerView

    android:id="@+id/recycle_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">

</android.support.v7.widget.RecyclerView>
```
可见此时，AppBarLayout与RecyclerView同级，而Toolbar被内嵌到了AppBarLayout中，RecyclerView加上了属性：

```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```
这个字符串是由DesignSupport库提供的，现在AppBarLayout就会随着RecyclerView的滑动事件做出反应了。

当然还有别的字符串，比如@string/bottom_sheet_behavior，用来实现底部栏，以后有机会再尝试吧。

看看目前的效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/AppBarLayout1.png)

现在，已经解决Toolbar被遮挡的问题。但这并不是AppBarLayout的全部作用，当AppBarLayout接收到滚动事件时，其实是可以**控制其子控件去响应这些事件**的。

现在Toolbar就是AppBarLayout的直接子控件，那么给Toolbar加上这样一段属性：

```
app:layout_scrollFlags="scroll|enterAlways|snap"
```
这里有三个属性：
1. scroll表示当RecyclerView向上滚动的时候，Toolbar会一起向上滚动，最后隐藏。
2. enterAlways表示当RecyclerView向下滚动的时候，Toolbar会一起向下滚动，重新显示出来。
3. snap表示当Toolbar还没有完全隐藏或者显示的时候，会根据当前滚动的距离自动选择隐藏还是显示。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/AppBarLayout2.png)

Toolbar随着RecyclerView的滚动而变化。

AppBarLayout的作用还没彻底展示出来，讲到CollapsingToolbarLayout的时候还会再用到它。

#### 9.SwipeRefreshLayout

- 下拉刷新的列表

一直以来无论是ListView、RecyclerView都没有自带刷新功能，更别说ScrollView了。现在support-v4库提供了一个布局：SwipeRefreshLayout

将列表镶嵌在这个布局中就能下拉刷新了，十分方便。看看布局代码：

```
<android.support.v4.widget.SwipeRefreshLayout
    android:id="@+id/swipe_refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycle_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.v4.widget.SwipeRefreshLayout>
```
简单的镶嵌，不过要注意的是，现在与AppBarLayout同级的是SwipeRefreshLayout了，所以layout_behavior这个属性要移到SwipeRefreshLayout上面了。

具体的刷新逻辑：


```
final SwipeRefreshLayout swipeRefreshLayout = findViewById(R.id.swipe_refresh);
    swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
        @Override
        public void onRefresh() {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                           initFoods();
                           adapter.notifyDataSetChanged();
                           swipeRefreshLayout.setRefreshing(false);
                        }
                    });
                }
            }).start();
        }
    });
```
代码不长，首先是调用了SwipeRefreshLayout的setOnRefreshListener方法，监听一个向下滑动的动作，然后在里面新开了一个线程，令线程沉睡2秒，不然太快看不出效果，然后又在里面使用了 runOnUiThread切换到主线程。

这里有一个全新的方法：**adapter.notifyDataSetChanged()**
这个方法和**setAdapter()**有什么区别呢？其实两者都能刷新列表，不过setAdapter由于是重新构造的adapter，列表会回到顶部，而notifyDataSetChanged()则会停留在当前位置。看具体场合来使用具体的方法吧。

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/SwipeRefreshLayout.png)

向下拉动RecyclerView的时候，顶部出现了一个图案表示正在刷新，刷新后列表的内容也改变了。

#### 10.CollapsingToolbarLayout

- 可折叠式标题栏

上次讲到AppBarLayout可以设置子控件对滚动事件的响应，不过里面只有Toolbar一个子控件。那么现在有一种更厉害的布局来作为AppBarLayout的子布局——CollapsingToolbarLayout。

CollapsingToolbarLayout可以将Toolbar跟其他控件包装成一个精美的标题栏。

1. 首先新建了一个Activity，名为FoodActivity，修改Adapter中的代码：

```
@Override
public void onBindViewHolder(@NonNull MyRecyclerViewAdapter.ViewHolder holder, int position) {
    final Food food = mFoodArrayList.get(position);
    holder.textView.setText(food.getName());
    Glide.with(mContext).load(food.getImageId()).into(holder.imageView);

    holder.cardView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent intent = new Intent(mContext, FoodActivity.class);
            intent.putExtra("Food_data", food);
            mContext.startActivity(intent);
        }
    });
}
```
这里其实使用了一下后面的知识，我将Food接上了Serializable接口，令Food类可序列化，这样就能用Intent传一个Food对象了。

再看看FoodActivity的布局文件：
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FoodActivity">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ToolbarTheme"
            app:contentScrim="@color/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/food_image"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin" />

        </android.support.design.widget.CollapsingToolbarLayout>

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <android.support.v7.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="15dp"
            android:layout_marginTop="35dp"
            android:layout_marginRight="15dp"
            android:layout_marginBottom="15dp">

            <TextView
                android:id="@+id/food_content_text"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_margin="10dp" />

        </android.support.v7.widget.CardView>

    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton

        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/message"
        app:layout_anchor="@id/appBar"
        app:layout_anchorGravity="bottom|end" />

</android.support.design.widget.CoordinatorLayout>
```
整体的布局：

1. CoordinatorLayout
    1. AppBarLayout
        1. CollapsingToolbarLayout
            2. ImageView
            3. Toolbar
    2. NestedScrollView
        1. CardView
            2. TextView
    3. FloatingActionButton

---

先梳理一下三个同级布局的关系。首先NestedScrollView要在AppBarLayout下方，而且AppBarLayout要子控件对NestedScrollView的滑动做出响应，因此NestedScrollView中加上属性：

```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```
FloatingActionButton在AppBarLayout的右下角，所以加上：

```
app:layout_anchor="@id/appBar"
app:layout_anchorGravity="bottom|end" 
```

---
再看AppBarLayout中的结构：

CollapsingToolbarLayout将ImageView和Toolbar包装了起来，然后设置属性：

```
app:layout_scrollFlags="scroll|exitUntilCollapsed"
```
**scroll**表示CollapsingToolbarLayout会随着列表滚动

**exitUntilCollapsed**表示CollapsingToolbarLayout折叠之后就保留在界面上，不再移出屏幕。

CollapsingToolbarLayout中有两个子控件：
1. ImageView：

```
app:layout_collapseMode="parallax"
```
parallax表示折叠过程中产生错位，带来一种视觉感官的提升。当然了，设成pin也是没关系的。
2. Toolbar：

```
app:layout_collapseMode="pin"
```
pin表示在折叠过程中位置始终保持不变。

这两个控件完美的组合在了一起。

---
剩下的结构都没什么好说的了，来看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/CollapsingToolbarLayout1.png)

- 完全展开时

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/CollapsingToolbarLayout2.png)

- 折叠后，ImageView和FloatingActionButton消失了

现在系统标题栏是蓝色的，与图片格格不入，不过这不是什么难事，轻易就能改的更美观。

在ImageView以及它所有的父布局中加入：

```
android:fitsSystemWindows="true"
```
改写一个style：

```
<style name="FoodTheme" parent="Theme.AppCompat.Light.NoActionBar">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorPrimary</item>
    <item name="android:statusBarColor">@android:color/transparent</item>
    <item name="android:textColorPrimary">#fff</item>
</style>
```
其中
```
<item name="android:statusBarColor">@android:color/transparent</item>
```
将状态栏的颜色指定为透明。

然后在Manifest中为FoodActivity指定为这个style：

```
<activity
    android:name=".FoodActivity"
    android:theme="@style/FoodTheme">
</activity>
```

就行了。看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/fitsSystemWindows.png)

Material Design的学习至此告一段落，今后App的制作将会一直使用Material Design来作为App的模板。