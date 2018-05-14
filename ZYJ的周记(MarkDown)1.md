# 张艺隽的 周记
>  刚开始使用markdown还不太会用，不过没关系了，慢慢熟悉吧。

### 4月10号: （主题：分享）
- 上网查了一个实现分享的办法，就是用++intent++。
1. new一个ComponentName对象。
2. intent.setComponent()，放入ComponentName对象
3. intent.setAction("android.intent.action.SEND")，非常重要，++设定intent的动作为发送++
4. intent.setType("text/plain")，非常重要，++设定发送的是图片还是文字++
5. putExtra放进去分享的内容
6. 当然是使用startActivity()了


```
ComponentName componentName = newComponentName("com.example.zhangyijun.testintent","com.example.zhangyijun.testintent.MainActivity "); 
Intent intent = new Intent();
intent.setComponent(componentName);
intent.setAction("android.intent.action.SEND");
intent.setType("text/plain");
String schedule = "[分享]++我的日程安排++：\n" + scheduleData.getDate() + "\n" + scheduleData.getTitle() + ":\n" + scheduleData.getContent();
intent.putExtra(Intent.EXTRA_SUBJECT, scheduleData.getTitle());
intent.putExtra(Intent.EXTRA_TEXT, schedule);
activity.startActivity(intent);
```

这里是自己实验一下用intent跳到另外一个程序的主活动里面,目标程序是自己写的。结果是跳转成功了。

ComponentName的构造器第一个参数是++目标包名++，第二个是++目标活动名++，这两个参数都可以从Manifest里面看的到，而现在要分享到微信和qq等，就要知道他们的包名和活动名，在网上找到了一个分享列表

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E5%88%86%E4%BA%AB1.png)

//*==TODO== 以后通过自己的代码来获取手机内所有可分享的目标软件*

### 4月11号: （主题：ListView排序）
- 使用++Comparable++排序ArrayList

1.  Comparable 是带有单一.compareTo()方法的接口
2.  一个实现了 Comparable 接口的类对象可以与其它同类型的对象进行比较
3.  实现 Comparable 接口的类需要重写 compareTo()方法，这个方法接收一个同类型的对象，并实现这个对象和传递给方法的另一个对象比较的逻辑。
4.  compareTo()方法返回Int类型的比较结果，分别代表下面的含义：
    1. 正值表示当前对象比传递给 comPareTO()的对象大
    2. 负值表示当前对象比传递给 comPareTO()的对象小
    3. 零表示两个对象相等


准备用这个东西来完成日程列表的排序方法。
首先把数组成员的类接上接口Comparable，此时提示要重写comparaTo()方法

~~具体代码如下：~~ 没有具体代码

- 因为这个方法只能比较一种属性，我现在要按修改时间和优先度两种属性排序的话只能通过改代码。
- 于是发现还有另外一个类：使用++Comparator++排序
- Comparator 接口与Comparable 接口相似，也提供了一个单一的比较方法叫作 compare()。
- 然而，与 Comparable的 compareTo()方法不同的是，这个 compare()接受两个同类型的不同对象进行比较。 
- 我将通过实现 Comparatoras 匿名内部类，对日程按照修改时间和优先度进行排序。于是具体代码如下：

```
public static Comparator eComparator = new Comparator() {
    @Override
    public int compare(Object o1, Object o2) {    
        ScheduleData scheduleData1 = (ScheduleData) o1;
        ScheduleData scheduleData2 = (ScheduleData) o2;
        if (scheduleData1.mEditTime < scheduleData2.mEditTime) {
            return 1;
        } else if (scheduleData1.mEditTime == scheduleData2.mEditTime) {
            return 0;
        } else return -1;
    }
};
```

调用如下：

```
Collections.sort(dataArrayList, ScheduleData.eComparator);
```
非常方便，这样排序就完成了。来看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E6%8E%92%E5%BA%8F1.png)

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E6%8E%92%E5%BA%8F2.png)

先按数字顺序从1到5建了五个日程，此时是正常的按增添顺序来排序。此时修改4的内容，现在：4排到最上面了。


### 4月12号: （主题：IO流）
- Java中I/O操作主要是指使用Java进行输入，输出操作。


#### 1. 什么是流？
>    1. Java所有的I/O机制都是基于数据流进行输入输出，这些数据流表示了字符或者字节数据的流动顺序。
>    2. 数据流是一串连续不断的数据的集合，就象水管里的水流，在水管的一端一点一点地供水，而在水管的另一端看到的是一股连续不断的水流。
>    3. 数据写入程序可以是一段、一段地向数据流管道中写入数据，这些数据段会按先后顺序形成一个长的数据流
 
几个小概念：
1. 数据流
一组有序，有起点和终点的字节的数据序列。包括输入流和输出流。
2. 输入流
程序从输入流读取数据源。数据源包括外界(键盘、文件、网络…)，即是将数据源读入到程序的通信通道。
3. 输出流
程序向输出流写入数据。将程序中的数据输出到外界（显示器、打印机、文件、网络…）的通信通道。

#### 2. 数据流的分类：
流序列中的数据既可以是未经加工的原始二进制数据，也可以是经一定编码处理后符合	某种格式规定的特定数据。因此Java中的流分为两种
1. 字节流：数据流中最小的数据单元是字节。
2. 字符流：数据流中最小的数据单元是字符，Java中的字符是Unicode编码，一个字符占用两个字节。 

顺便看了一下++字符流和字节流的区别++。

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/IO1.png)

#### 3. IO流的分类：
###### 这么多流，应该选什么？
1. **确定是数据源还是数据目的（即输出还是输入）**

源 | 目的
---|---
InputStream | Reader
OutputStream | Writer

2. 明确操作的数据是否是纯文本

是（字节流）| 否（字符流） | 转换
---|---|---
Reader|InputStream|InputStreamReader等
Writer|OutputStream

3. 明确具体的设备
    
硬盘文件：
    
FileInputStream| FileOutputStream
---|---
FileReader |FileWriter
    
内存用数组：
    

byte[] | char[]|String
---|---|--- 
ByteArrayInputStream| CharArrayReader|StringReader
ByteArrayOutputStream|CharArrayWriter |StringWriter
网络：Socket流
    
键盘：System.in、System.out

4. 是否需要缓存提高效率
是就加上Buffered

- 补充一张图片：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/IO2.png)




**学习了流怎么能不用呢？接下来我就要使用流来实现联网获取日历的资料~**

### 4月13号: （主题：联网）

 - 不用第三方库OkHttp来实现联网功能，使用最基础的HttpURLConnection
 
先看下做完的结果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E8%81%94%E7%BD%914.png)

还能查看节假日和节气。

[这是我找的API接口，免费而且是JSON格式的](https://blog.csdn.net/oqqsoso123456/article/details/72782353)，非常感谢这位博主。

##### 接下来就是怎么实现了：
 
 使用HTTP协议访问网络：
1. 首先要new出一个URL对象，并传入目标的网络地址，然后使用.openConnection()方法即可获得HttpURLConnection实例：

```
URL url = new URL(“http://www.baidu.com”);
HttpURLConnection connection = (HttpURLConnection) url.openConneciton();
```


2. 然后设置一下HTTP请求所使用的方法，主要有GET和POST

```
connection.setRequestMethod("GET");    
```

**此处网站指明使用什么格式的参数和请求所使用的方法：**

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E8%81%94%E7%BD%911.png)

3. 之后用getInputStream()方法就可以获取到服务器返回的输入流了:

```
InputStream in = connection.getInputStream();
```

4. 最后connection.disconnect();关闭连接：

```
connection.disconnect();
```

5. 对流进行读取：
    1. 是输入流，所以选：InputStream、Reader
    2. 要转换，所以选：InputStreamReader
    3. 要提高效率，所以选： BufferedReader

```
reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
```
~~装逼写法~~

然后就是把读出来的数据存放到字符串里面：

```
StringBuilder respone = new StringBuilder();
String line;
While((line = reader.readline())!=null){
Respone.append(line);
}
```

- 这里 ++readline()++ 方法是读取流数据的时候用的，当读到换行标记时就会跟着换行，同时以字符串形式返回这一行的数据，当读取完所有的数据的时候会返回null
- 所以while中的表达式==null时表示已经读完了，退出循环

6. 这里网站提供的是JSON格式的数据，这种格式非常简单

```
JSONObject jsonObject = new JSONObject(jsonData);
//jsonData即上面存放好的字符串
```

然后使用JSONObject的各种get方法即可。

- 有两种特殊的情况：
---
一是这种：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E8%81%94%E7%BD%912.png)

这是一个数组。New一		个JSONArray对象，用getJSONObject(index);方法即可

```
JSONArray FestvialList = jsonObject.getJSONArray("festivalList");
```
---

二是这种：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E8%81%94%E7%BD%913.png)

一个JSONObject中包含了另一个JSONObject名为:”jieqi”，此时只要再实例化这个JSONObject			
然后调用这个新的JSONObject的get方法即可

```
JSONObject jieQi = jsonObject.getJSONObject("jieqi");   
```
---

**至此，联网功能基本实现了！**

### 4月14号: （主题：图片）

- 从相册中选取图片

其实很简单：
1. 运行时权限（Manifest里面注册，还要在运行时向用户请求权限）

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

```
if (ContextCompat.checkSelfPermission(activity, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED){
    ActivityCompat.requestPermissions(activity, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
}
```
- 第一句是查看是否已经申请权限，若没有，执行第二句来申请权限
2. 书上写的又复杂又难懂，其实一句不就搞定：

```
Intent intent = new Intent(Intent.ACTION_PICK,MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
```

3. 重写活动的onActivityResult(int requestCode, int resultCode, Intent data)，调用这个对象data的.getData();方法获得一个Uri对象：

```
Uri selectImage = data.getData();
```

4. 然后用getContentResolver().query这个方法获得一个Cursor对象：

```
Cursor cursor = getContentResolver().query(selectImage,filePathColumn,null,null,null);
cursor.moveToFirst();

```
- 第二个参数是本地图片媒体路径

```
String[] filePathColumn = {MediaStore.Images.Media.DATA};
```

5. getString获得这个图片的地址：

```
String path = cursor.getString(cursor.getColumnIndex(filePathColumn[0]));
```

6. 关闭cursor(老是不记得)

-  图片的地址最终会以text的类型**存放**在数据库中对应日程的地方，如果一个日程有多张图片，那么，他们的地址是用逗号连接起来的：

 ```
 if (pic_path.isEmpty()) { 
     pic_path = path;
 } else {
     pic_path = pic_path + "," + path;
 }
 ```

- 在**显示**图片的时候用.split("分隔符")方法，以逗号为分割符来分割图片地址，然后加载每一张图片出来:

```
String[] spath = pic_path.split(",");
```

- 在**删除**图片的时候用contains和replace方法，把整个字符串中该图片的地址删掉。不得不说java太好用了，想干什么都能直接用现成的方法。用c的时候处理字符串贼难受。代码如下：

```
while (allPath.contains("," + pic_path) || allPath.contains(pic_path)) {    //删去整个地址字符串中此图片的地址字符串
    if (allPath.contains("," + pic_path)) {
        allPath = allPath.replace("," + pic_path, "");
    } else if (allPath.contains(pic_path)) {
        allPath = allPath.replace("" + pic_path, "");
    }
}
if (!allPath.isEmpty()) {                                                   //避免地址中第一个字符是逗号的情况
    if (allPath.substring(0, 1).equals(",")) {
        allPath = allPath.replaceFirst(",", "");
    }
}
```

当然还要注意把分隔符逗号删掉，不然程序在没有找到图片地址、也没有做异常捕捉的情况下程序会崩掉。

删后的地址重新update到数据库就行了

通过各种看资料，写出了一个缩小Bitmap图的类，还有一个可以旋转图片的按钮。具体代码就不看了。

```
Matrix matrix = new Matrix();
matrix.setRotate(90f); //旋转90°
mBitmap = Bitmap.createBitmap(mBitmap, 0, 0, mBitmap.getWidth(), mBitmap.getHeight(), matrix, true);
```

现在App可以放入图片了哦~还可以旋转和删除，来看看效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E5%9B%BE%E7%89%871.png)

旋转：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E5%9B%BE%E7%89%872.png)


### 4月14号: （主题：界面）

- 修改一下界面

**今天大换血了**，把42个显示日期的TextView控件全删了。在代码里面动态加载TextView，不然layout文件看着++太傻了++，而且也不能++适配各种大小的屏幕++，而且每次打开layout文件就++卡的一批++。

其实也有不好的地方，为了适配屏幕，控件的宽就要随屏幕大小变化。那么背景图可能会出现拉伸现象。我只能用下面这个办法，令每个控件变成长宽相同的正方形：

```
int height = display.getWidth();
LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(0, height/7, 1.0f);
```

但是这样TextView布满了整个屏幕，覆盖了父布局的监听事件，那么又得另外找办法。

改了一下主界面，是不是好看了点。而且日程左边的颜色条是透明的：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/UI.png)

### 4月15号: （主题：Git）

- 使用git和github

Github.com

几个概念：
- Repository仓库 一个项目对应一个仓库
- Star收藏
- Fork克隆仓库 forked from 别人的仓库
- Pull request发起请求，合并代码
- Watch 关注项目，项目更新时获得通知
- Issue 讨论bug

**Start a project:**

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/github1.png)

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/github2.png)

##### Git本地仓库：

填用户名，还要去某个地方填下密码，git才能实现上传到github
1. git config --global user. name ’MightyZYJ’
2. git config --global user. name ’1123434219@qq.com’
3. git init 初始化 
4. git status 查看状态
5. git add 添加到暂存区
6. git commit -m  ‘描述’ 添加到本地仓库 
7. git push 推送到github仓库

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/git.png)

提交到本地仓库。

### 4月16号: （主题：总结）

- 一个星期就这样过去了……

时间过的真快啊，不知不觉写了十几个类了

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/%E5%A5%BD%E5%A4%9A%E7%B1%BB.png)

现在APP的功能已经100%实现了，但是我并不满意，因为一开始想着快点把APP做完，然后再把Java和Android彻底过一遍。现在想了想，Java我只是学会了皮毛，更别说Android了，还是要多学多做，不要单纯图个快，那样什么都学不好。剩下的时间继续看书打码。
> 积累多点经验，继续优化自己的代码，不要止步于做完APP这个水平。                        --鲁迅
