# 张艺隽的 周记
>  

### 4月17号: （主题：Socket）

- 既然学了IO流，那么加以利用，配合Socket实现客户端之间传送信息

#### 1.什么是Socket？
> Socket即套接字，是一个对 TCP / IP协议进行封装 的编程调用接口（API）
  
#### 2.为什么要用Socket？
> 因为通过Socket，我们才能在Andorid平台上通过 TCP/IP协议进行开发

#### 3.Socket有什么使用类型？
> - 流套接字（streamsocket） ：基于 TCP协议，采用 流的方式 提供可靠的字节流服务
>- 数据报套接字(datagramsocket)：基于 UDP协议，采用 数据报文 提供数据打包发送的服务

![image](http://upload-images.jianshu.io/upload_images/944365-8df0ed7afe6b32d1.png)

说到UDP，我很喜欢一个比喻：
> UDP就像写信，在信封写上收信人名称、地址就可以交给邮局发送了，至于能不能送到，就不关我事了!!

#### 4.Socket和Http有什么区别？
- Socket属于传输层，因为 TCP / IP协议属于传输层，解决的是**数据如何在网络中传输**的问题
- HTTP协议 属于 应用层，解决的是**如何包装数据**

#### 5.Socket实战！

> 先在Eclipse上面试试手

 1. **首先建立一个server**：
 
```
ServerSocket server = new ServerSocket(8088);
```
这句话是什么意思呢？
- 在ServerSocket()构造器中输入端口8088
- 然后在本机的IP下用这个端口建立一条Socket
    - 那么这句话又又又是什么意思？
        - 首先IP地址是IP协议提供的一种统一的地址格式
        - 如果把IP地址比作一间房子，那么端口就是出入这间房子的门。真正的房子只有几个门，但是一个IP地址的端口可以有65536之多！
        - **结论：** 这句话的意思就是利用Socket通信，必须要知道服务器的ip地址和端口

- ##### 太好了！这样我们就不局限于本地传送消息了。
    - 注意几点：
        1. IP是动态分配的，自己试过每次开校园网ip都不一样
        2. 端口不是乱填都可以的，1000以内还有附近的端口都给系统某些软件占用了。

 2. **实例化Socket**：
 
```
Socket socket = server.accept();
```
相当于现在有了一条只有起点而没有目的地的通道，这条通道谁连上都可以。

 3. **读取信息**： ~~又到了装逼写法的时刻~~

```
BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));  
```
这里先实现从客户端接收信息吧。还是传统的三步走：
- 字节流，输入流：InputStream
- 转换流：InputStreamReader
- 提高一下效率： IufferedReader

 4. **展示信息**：
 
```
System.out.println(br.readLine());
```
**那么一个server已经建好了，可是，要知道客户端是无时无刻发送信息过来的。总不能收到一条信息就把程序关掉吧。于是我加上一个死循环来保证客户端一直工作。**

```
while(true) {
	Socket socket = server.accept();
	BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));  
	System.out.println(br.readLine());
}
```
**死循环里面"一个accept一个客户端"**

 5. **实现客户端**：

```
Socket client = new Socket("localhost", 8088);
```
"localhost"就是本机地址了，8088是前面写的端口，两个参数都对上了才能连上那条没有目的地的通道。现在通道已经对接上了，可以来实现数据的传递了。

 6. **输入信息**：
 
```
if(client.isConnected()) {
	BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()));
	bw.write("<张艺隽>connected to this computer");
	bw.newLine();
	bw.flush();
}
```
不废话了，全部贴上来。
- 第一行，标准写法
- 第二行，写入内容
- 第三行，**划重点了**，为什么要newLine()呢？因为server里面读信息是用BufferedReader的readline()，这个方法以行结束符为终点，将沿途的字符输出。如果没有行结束符，那么将不能正常读出数据。这里在内容中加上\n也是可以的。
- 第四行，释放内容

先运行server，再运行client，来看看结果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/socket1.png)

可以看到我们的server已经收到来自client的信息了。

#### 6.Socket实现文件传输

捋一捋思路
1. 要用socket传输文件，首先要有File对象、源头和目的地
2. server选择输入流，client选择输出流
3. 进行输出和读取
4. 释放资源

##### 1. File对象
在client中：
```
File src = new File("C:\\Users\\11234\\Desktop\\testFile\\github1.png");
```
在server中：


```
File des = new File("C:\\Users\\11234\\Desktop\\testFile\\github1.png");
```
先把路径写死，以后再改。练手，写简单点，分隔符也不用了。

##### 2. 选择流
client中：
```
InputStream fis = new FileInputStream(src);
OutputStream os = new DataOutputStream(client.getOutputStream());
```


server中：

```
InputStream is = new DataInputStream(socket.getInputStream());
OutputStream fos = new FileOutputStream(des);
```


##### 3. 输出

```
byte[] bytes = new byte[1024];
int len = 0;
while(-1 != (len = is.read(bytes))){//边读边写
	os.write(bytes,0,len);
	os.flush();
}
```

##### 4. 释放资源
- 先打开的后关闭
```
os.close();
fis.close();
```
```
fos.close();
is.close();
```
##### 5. 运行查看结果
先运行server，再运行client

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/socket2.png)

成功了！下一步，创建双向的通道。

##### 6. 双向传输

> 这里跳跃有点大，因为做的时候出现了一点bug，还有一段封装类的过程，所以没有记录下来。

- 思路
    1. 消除client和server的差异，因此我把发送和接收封装成两个类，client和server就能轻松调用。
    2. 主线程中用死循环不断监听是否对方有发送文件的请求。若要主动发送文件，则新建子线程，在子线程中进行发送文件的操作。

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/socket3.png)

目前的效果图，可以实现远程文件的发送和接收。本来这个东西一天就做完了最基本的功能，但是后面加入多线程的时候出了很多问题。比如：用户有**发送**和**接收**两个线程，**发送**的线程里面需要readUTF来判断对方是否接收文件，但是**接收**的线程里面又要不断readUTF来判断对方是否申请发文件过来。这时两个线程都抢着读取流，但是流里面的内容是一次性的，也就是read完就没有了。解决了这个问题之后再转战Android吧。目前的思路有线程池、队列

### 4月18号: （主题：日期时间选择器）

Android5的时代已经过去了，但是它刚面世时带来了许多非常漂亮的控件。其中**DatePickerDialog**和**TimePickerDialog**就是今天我要使用的控件。

在一般的日程管理APP中，用户可以自行选择哪一年、哪一月、哪一天，而不用一直滑动屏幕。既然有这个需求，那么我就做出来。
#### DatePickerDialog:

直接放代码：

```
new DatePickerDialog(activity, new DatePickerDialog.OnDateSetListener() {
    @Override
    public void onDateSet(DatePicker view, int year, int month, int dayOfMonth) {
        Calendar calendar = Calendar.getInstance();
        calendar.set(year,month,dayOfMonth);

        SlideScreen.slideScreen(activity,creater,refresher,changer,calendar);

    }
},today.get(Calendar.YEAR),today.get(Calendar.MONTH),today.get(Calendar.DAY_OF_MONTH)).show();
```
就像一般的dialog控件一样，记得.show()就好了。DatePickerDialog的构造器中，第一个参数是Activity,第二个参数是一个监听器，要求重写其中的onDateSet方法，这个方法会在用户选择日期之后调用，返回了三个一看就明白的参数，就是用户选择的年月日。不得不说我的代码封装的太好了，这里直接用滑动屏幕那个类的方法，介绍一下这个我写的类中要传入的四个参数：
1. Activity对象 这个不用说了
2. CreateCalendar对象，这个是我用来构建日历内容的
3. RefreshList对象，这个用来刷新列表的
4. ChangerTop对象，这个用来改变日历头顶部分的
5. Calendar对象，将用户选择时间后得到的年月日处理成Calendar对象，然后传入此处。

试验一下效果：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/DatePickerDialog.png)

当我想办法把里面的内容改成中文时，才发现只要系统是中文里面的文字就是中文的，智能啊。

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/DatePickerDialog2.png)

#### TimePickerDialog:

是否有这样的需求：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/TimePickerDialog.png)

在某一天，班级活动定在了下午三点，可是时间和地点都得手动输入，是否太麻烦了。那我的APP需要做到精确到何时何秒吗，答案是肯定的。而且现在有TimePickerDialog这个如此好用方便的控件。

看看这个TimePickerDialog是怎么样的：

![image](https://raw.githubusercontent.com/MightyZYJ/MyCalendar/master/TimePickerDialog2.png)

也是非常的美观，下面贴代码：

```
new TimePickerDialog(activity, new TimePickerDialog.OnTimeSetListener() {
    @Override
    public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
        String time = String.valueOf(hourOfDay)+" : "+String.valueOf(minute);
        showTime.setText(time);
    }
},0,0,true).show();
```
依旧是非常的简单，用户选择时间后调用onTimeSet这个方法，这里把时间显示出来就行了。后续工作把这个时间存到数据库里面就行了。

### 4月21号: （主题：Queue）

- 实在没啥好写了，干脆写一下Queue吧

队列分为单向队列和双向队列。
1. 单向队列
    - 队列通常FIFO（先进先出）
    - 优先级队列和堆栈LIFO（后进先出）


 操作  | 抛出异常 | 特殊值
---|---|---
插入 | add(e) | offer(e)
移除 | remove() | poll(e)
获取 | element()| peek()

**ArrayQueue**是Queue接口的一个实现类。

这里我们先定义一个接口Request,接口内有一个未实现的方法void deposit();模拟的是人们在银行排队存款的情景。↓
```
interface Request{
    void deposit();
}
```
这个方法遍历了整个队列，将队列中的Request对象出队，并调用它们的desposit()这个方法，即我们重写的模拟存款的方法。↓
```
public static void dealWith(Queue<Request> queue){
    Request man = null;
    while(null != (man = queue.poll())){
        man.deposit();
    }
}
```
这是主要的函数，从0到9进队了10个人，每个人都重写了他们的desposit()方法。↓
```
Queue<Request> queue= new ArrayQueue<Request>;
for(int i = 0 ; i < 10 ; i++) {
    final int num = i;
    queue.offer(new Request(){
    
        @Override
        public void deposit(){
            loge(TAG,"第"+num+"个人，存款数额："+Math.random()*10000);
        }
    });
}
dealWith(queue);
```

2. 双向队列
    - double-ended queue ，是一种具有队列和栈的性质的数据结构。双端队列中的元素可以从两端弹出，限定插入和删除操作在表的两端进行。
    - 可用做队列和堆栈

这里用双向队列实现自定义堆栈：


```
private Deque<> container = new ArrayDeque<E>();
private int cap;

//构造器，传入容量
public MyStack(int cap){
    super();
    this.cap = cap;
}

//压栈
public boolean push(E e){
    if(container.size() + 1 > cap){
        return false;
    }else{
        return container.offerLast(e);
    }
}

//弹栈
public E pop(){
    return container.pollLast();
}

//获取
public E peek(){
    return container.peekLast();
}

```

其实非常容易理解，就是把双向队列的头给封死了，以达到栈的LIFO。这里的cap即栈的容量。
1. offerLast()是双向队列中的入队方法，返回一个boolean值，这里巧妙的把这个方法放到了return中。这里还写了一句
    ```
        if(container.size() + 1 > cap){return false;}
    ```
    用来人为规定队列的长度。
2. pollLast()也是双向队列中的方法，把这个队列最后一个元素踢出队伍，并返回这个元素。
3. peekLast()就和pollLast()比较相似了，不过只返回队尾元素，不会把元素踢出来。

这样就实现了一条LIFO的队列，顺便还复习一波泛型。和C语言相比简直太舒服了。

==//TODO==能不能用队列来解决多线程Socket中，流被各子线程抢用的情况呢？




### 4月22号: （主题：总结）

哇又是过的贼快的一个星期。周末学习队列花了太多时间了！如果有更多时间就好了（+1s），日程管理的APP也没怎么动过，本来还有很多改进的空间的。下个星期迎来一波期中考试？不过还是得再抽出更多时间看java和Android。

列一下目前最想看的的内容：
1. RecyclerView，这个想看很久了
2. Fragment，久仰大名
3. MessageQueue，很想解决我在Socket中发现的那个问题
4. DesignSupport库，想优化我APP的界面
