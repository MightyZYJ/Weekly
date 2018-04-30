### 4月24号: （主题: RecyclerView）

- 终于开始学习RecyclerView了

在之前的开发中，曾领略过ListView的功能，也体会到ListView不够强大的扩展性，它只能实现数据纵向滚动的效果，无法做到更美观更神奇的效果。

为此，Android提供了**RecyclerView**，这是一个功能强大的列表控件，能实现**横向滚动、网格列表、瀑布流列表**。基于我边学边做的风格，我打算使用RecyclerView来实现网格布局，由此取代之前在日历中动态添加TextView的这个思路。

##### 1. 在build.gradle包中加入依赖库

```
implementation 'com.android.support:recyclerview-v7:27.1.1'
```

##### 2. 在布局文件中添加RecyclerView

```
<com.example.zhangyijun.mycalendar.widget.MyRecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_below="@id/Calendar_top">
</com.example.zhangyijun.mycalendar.widget.MyRecyclerView>
```
- 这里加入的是我写的一个继承于RecyclerView的类，为什么要这样做，一会再说。


##### 3. 准备适配器

就像ListView一样，RecyelerView也需要适配器。

```
public class MyRecyclerAdapter extends RecyclerView.Adapter<MyRecyclerAdapter.ViewHolder> {

    private ArrayList<Integer> mDayList;

    public static class ViewHolder extends RecyclerView.ViewHolder {
    
        TextView textView;
    
        ViewHolder(View itemView) {
            super(itemView);
            textView = itemView.findViewById(R.id.item_text);
        }
    }

    public MyRecyclerAdapter(ArrayList<Integer> list;) {
        this.mDayList = list;
    }


    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {

        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_layout, parent, false);
        ViewHolder holder = new ViewHolder(view);

        return holder;
    }



    @Override
    public void onBindViewHolder(@NonNull final ViewHolder holder, int position) {
        ...
    }

    @Override
    public int getItemCount() {
        return mDayList.size();
    }
}
```
1. 首先定义了一个内部类ViewHolder，继承至RecyclerView.ViewHolder。然后在他的内部定义了一个构造函数，这个构造函数中传入了一个View对象，通过这个View对象就能使用findViewById了。
2. 重写OnCreateViewHolder这个方法，动态加载子项的布局，然后作为View参数来构造ViewHolder对象，返回这个对象。
3. 重写onBindViewHolder方法，这个方法在子项滑入屏幕中时会被调用，在这里我写了子项的一些处理事件，代码太长就省略了。
4. 重写getItemCount方法，要在里面返回列表的长度。

一个最基本的Adapter就写好了，然后我又写了一个方法，将Calendar对象转换为Integer列表，然后把列表传入我的Adapter的构造方法中就好了。

现在为了实现网格布局，有这样的方法：

```
GridLayoutManager gridLayoutManager = new GridLayoutManager(activity, 7);
recyclerView.setLayoutManager(gridLayoutManager);
```
日历是7列的，所以这里传入第二个参数是7。这样就做好了，看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/RecyclerView1.png)

为了解决以前遇到的父布局监听与子控件监听冲突的问题，我把滑动切换月份和子项的onTouch事件放在了一起。这时候出现了一个小问题，就是RecyclerView**把上下滑动的动作给拦截了**。因此想要滑动切换月份，需要**手指非常平直地左右滑**，稍微向上偏一点，这个动作就无法传递到子控件中。

我当然不能忍，于是自己写了一个MyRecyelerView继承自RecyelerView，然后在里面重写了onInterceptTouchEvent方法。
```
@Override
public boolean onInterceptTouchEvent(MotionEvent e) {
    return false;
}
```
这样做其实是我纯属想试试而已。当我在运行一遍的时候发现真的可以！！上下滑动的动作也不会被拦截了。

> Android学得越久，越感觉积累的重要性。

### 4月25号: （主题: ViewPager）

- ViewPager，是什么来的？

ViewPager常用于各种APP的开启引导界面，例如京东：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/ViewPager1.png)

使用的就是一个ViewPager。虽然书上没有这一节，但是抱着好奇的态度，我寻找了相关的教程。

使用ViewPager有几个步骤：

##### 1.编写PagerAdapter
ViewPager像ListView、RecyclerView一样需要一个适配器。自定义一个类，继承自PagerAdapter。

首先创造一个构造方法：

```
private ArrayList<View> viewArrayList ;
```


重写其中的四个方法：

```
@Override
public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    container.removeView(viewArrayList.get(position));
}

@NonNull
@Override
public Object instantiateItem(@NonNull ViewGroup container, int position) {
    container.addView(viewArrayList.get(position));
    return viewArrayList.get(position);
}

@Override
public int getCount() {
        return viewArrayList.size();
}

@Override
public boolean isViewFromObject(@NonNull View view, @NonNull Object object) {
    return (view == object);
}
```
1. **destroyItem()**:ViewPager中不可能保留所有的Item，所以在他们不需要的时候，可以将他们移除，因此重写这个方法。使用参数container的removeView方法，将指定位置的Item移去。
2. **instantiateItem()**:这个方法其实跟ListView中的getView()方法、RecyclerView中的onBindViewHolder()有异曲同工之妙。
在里面使用参数container的addView方法，将Item加入到当前显示的Page上。
3. **getCount()**:这个不用多说了吧。
4. **isViewFromObject()**:这个方法来判断当前的View是不是我们需要的一个对象。

##### 2. 布局文件中声明


```
<android.support.v4.view.ViewPager
    android:id="@+id/viewpager"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</android.support.v4.view.ViewPager>
```
##### 3. 在主函数中调用ViewPager

```
private void initViews(){
    LayoutInflater inflater = LayoutInflater.from(this);
        
    ArrayList<View> viewArrayList = new ArrayList<>();
    //View view = inflater.inflate(R.layout.page_one,null);
    //ActivityMap.viewPager = view;
    //viewArrayList.add(view);
    viewArrayList.add(inflater.inflate(R.layout.page_one,null));
    viewArrayList.add(inflater.inflate(R.layout.page_two,null));
        
    ViewPagerAdapter adapter = new ViewPagerAdapter(viewArrayList);
        
    ViewPager viewPager = findViewById(R.id.viewpager);
    viewPager.setAdapter(adapter);
    }
```
代码很简单，利用LayoutInflater对象，将两页布局加到了数组里面，把这个数组传入PagerAdapter的构造函数中，最后setAdapter就好了。运行程序。好吧开门红，一片bug。

看了一下error的描述，各种地方都出现了空指针。出现error的地方一般都用了findViewById。那么真相就大白了，findViewById这个方法，默认调用时应该是：

```
this.findViewById();
```
由于很多控件是动态生成在ViewPager里面的，因此不能直接用findViewById()了。

既然知道了原因，那么就好办了。直接先通过LayoutInflater对象来实例化布局。然后再**用这个view的findViewById()** 就行了。
```
Viewview = inflater.inflate(R.layout.page_one,null);
listView = view.findViewById(R.id.listView);
```

看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/ViewPager2.png)

可以左右滑动翻页了。但是我并不希望左右滑动翻页，因为和日历部分的左右滑动切换月份的监听冲突了。不过自从上次继承RecyelerView后，现在已经无所畏惧了，直接继承ViewPager，重写onInterceptTouchEvent()方法吧！

其实ViewPager滑起来有一个惯性的滑动翻页效果，用来做日历的滑动切换月份十分不错。但是才刚用完网格RecyclerView又要换到ViewPager。。emmmmm

现在实现点击按钮来翻页。

```
private void viewPagerButton() { 
    Button changePageButton = findViewById(R.id.page_button);
    final MyViewPager myViewPager = findViewById(R.id.viewpager);
    changePageButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            int page = myViewPager.getCurrentItem();
            myViewPager.setCurrentItem(1 - page, true);
        }
    });
}
```
真是被我的机智折服。因为只有0和1两页，所以先获取当前的page，然后跳到1-page这页，就实现了从0翻到1、从1翻到0了。


### 4月29号: （主题：总结）

现在APP的样子：

（手机截图出来的图片真的好大啊）

（主界面）

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/end1.png)

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/end2.png)

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/end3.png)

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/end4.png)

> 这就是APP目前的样子了，还会修改，会再添加一些新功能。
> 
> 目前APP先放一放了，花多点时间和精力再进修一下JAVA和Android。
> 

