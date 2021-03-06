### UNIX下IO模型

1.同步和异步

同步与异步是线程之间的关系，两个线程同步操作时，调用者需要等待被调用者返回结果，之后进行下一步操作，异步则是不需要等待返回结果，即可进行下一步操作，被调用者通常依靠事件，回调等机制来通知调用者返回的结果。

2.阻塞和非阻塞

阻塞和非阻塞是同线程内的关系，一个线程在某个时刻，要么是阻塞的，要么是非阻塞的。
阻塞是指调用结果返回之前，当前线程被挂起，得到返回结果之后才可以继续执行。
非阻塞是指该线程在结果未返回之前，不会被挂起，可以继续执行。

四种组合：

同步阻塞：发送方发送请求之后一直等待响应，接收方处理请求，在得到返回结果之前将线程挂起。

同步非阻塞：发送方发送请求后一直等待响应，接收方处理请求，在得到返回结果之前，可以继续做其他的事情，发送方一直等待。等待发送完成之后，将状态和结果返回接收方，接收方再响应发送方	。

异步阻塞：发送方发送请求之后，不等待响应，可以继续干其他的事，接收方处理请求，再得到返回结果之前，一直等待结果返回，返回之后再响应发送方。

异步非阻塞：发送方发送请求之后，不等待响应，可以继续干其他的事，接收方处理请求，在得到结果之前，也可以干其他的事，当IO操作完成之后，将完成的状态和结果通知接收方，之后响应发送方

|IO操作分为两个阶段：
|1.数据准备阶段     2.内核空间复制数据到用户空间阶段

Unix下的五种IO模型：

阻塞IO：一个进程recvfrom开始IO之后，它的系统调用直到数据包到达且被复制到用户空间或者中间发生了错误才会返回。进程从调用recvfrom开始到它返回成功这整个过程都是阻塞的。如图：

![阻塞IO](pic/阻塞IO.png)



非阻塞IO：从recvfrom调用开始，内核如果无数据报准备（缓存没有数据），就会返回一个EWOULDBLOCK错误，然后recvfrom一直检查有无数据到来。(一般不使用)

![非阻塞IO](pic/非阻塞IO.png)



IO复用：IO复用与非阻塞IO有点像，因为非阻塞IO会花费大量的CPU资源去轮询，而且可能有很多的任务同时进行，人们就想到了循环查询多个任务的完成状态，有一个任务完成，就去处理这个任务。Unix下的select，poll和epoll就是做这个事情的，netty就是基于epoll实现的。

![IO复用](pic/IO复用.png)

信号驱动IO：首先允许开启套接口信号驱动IO功能，通过系统调用sigaction执行一个函数，调用之后立即返回，进程继续执行，当内核中数据报准备好之后，会为该进程生成一个SIGIO信号，告诉进程准备好了，可以来读取数据了。

![信号驱动IO](pic/信号驱动IO.png)

异步IO：告知内核我要进行IO操作，并让内核在整个操作完成之后通知我们，在内核准备和复制数据报期间，进程继续执行。这个和上面的信号驱动相比主要区别是，信号驱动告诉进程什么时候可以进行IO操作，而异步IO是告诉进程IO操作何时完成的。

![异步IO](pic/异步IO.png)

###select，poll，epoll

非阻塞IO，进程会时刻轮询来达到处理多个流的目的，但是如果所有的流中都没有数据，那么就会白白的浪费CPU，为了避免CPU空转，就引进了一个代理select，它可以同时观察许多流的IO事件，进程会阻塞在select，当有一个或者多个流有IO事件时，select就会醒来，把所有的IO都轮询一遍，进行IO操作。使用select，事件复杂度就是O(n)，32位默认连接数是1024，64位默认是2048。

poll和select类似，将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态。poll没有最大连接数的限制，因为它是基于链表来存储的。它的缺点是大量的数组从用户到内核之间复制，不管这样有没有意义。

epoll则是在select和poll基础上进行了优化，没有最大连接数限制，只有某个IO流有数据，这个IO流就会callback调用它的进程，这样就不用进行轮询，时间复杂度就是O(1).

epoll是通过epoll_create,epoll_ctl和epoll_wait三个接口实现的。

**epoll_create**是在内核创建一个epoll相关的列结构，并且将fd返回用户态。

**epoll_ctl**是将fd添加/删除在epoll_epfd中，epoll_event是用户态和内核态交互的结构，定义了用户态的事件类型和数据载体epoll_data。也就是告诉内核哪些fd需要做哪些操作。

**epoll_wait**是阻塞等待内核返回的可读写事件，epfd还是epoll_create的返回值，events是结构体指针存储epoll_event,就是讲所有待处理的epoll_event结构存储下来，maxevents告诉内核本次返回的最大fd数量，这个和events指向的数组是相关的。也就是调用epoll_wati等待内核反馈的可读写事件并处理。

### Java IO

java io就是输入输出，从数据源读取以及输出到目标媒介。常见的数据源和目标媒介就是：文件，管道，网络连接，内存缓存等。

##### 流

Java IO中，流是一个核心概念，流是指一个连续的数据流，既可以从流中读取数据，也可以写入数据。在Java IO中，流可以是字符流（以字符为单位），也可以是字节流（以字节为单位）。

##### Java IO的用途

Java IO包含了InputStream，Reader，OutputStream，Writer以及它们的子类，每一个类提供不同的功能。这些类的主要作用就是：文件访问，网络访问，内存缓存访问，线程内部通信，缓冲，过滤，解析，读写文本，读写基本数据类型，读写对象。

![QQ截图20141020174145](pic/QQ截图20141020174145.png)

### Java NIO

NIO是(Non-block I/O),就是非阻塞IO。它提供了两种新的套接字ScoketChannel和ServerSocketChannel，这两支持阻塞和非阻塞模式，阻塞编码简单，但是性能和可靠性不好，非阻塞模式相反。

核心类库：

1. 缓冲区Buffer

   Buffer是一个对象，实质上是一个数组，提供了对数据的结构化访问和维护读写位置等信息。在NIO库中，所有数据处理都是通过缓冲区的(W/R)。每一种Java基本数据类型都对应一种缓冲区，最常用的就是ByteBuffer缓冲区。

   - ByteBuffer：字节缓冲区
   - CharBuffer：字符缓冲区
   - ShortBuffer：短整型缓冲区
   - IntBuffer：整型缓冲区
   - LongBuffer：长整型缓冲区
   - FloatBuffer：浮点型缓冲区
   - DoubleBuffer：双精度缓冲区

2. 通道Channel

   Channel是一个通道，网络数据通过Channel读取和写入。它与流的功能类似，但是它是双向的，可以读、写或者二者同时进行。

3. 多路复用器Selector

    选择器用于监听多个通道的事件（比如：链接打开，数据到达）。因此，单个线程可以监听多个数据通道。 

##### NIO服务端序列图

![nio服务端序列图](pic/nio服务端序列图.png)

```java
// 1.打开ServerSocketChannel
ServerSocketChannel acceptSvr = ServerSocketChannel.open();
// 2.绑定监听地址InetSocketAddress并设置非阻塞模式
acceptorSvr.socket().bind(new InetSocketAddress(InetAddress.getByName(“IP”), port));
acceptorSvr.configureBlocking(false);
// 3.创建Selector，启动线程
lector selector = Selector.open();
New Thread(new ReactorTask()).start();
// 4.将ServerSocketChannel注册到Selector，开启监听ACCEPT事件
SelectionKey key = acceptorSvr.register( selector, SelectionKey.OP_ACCEPT, ioHandler);
// 5.Selector轮询就绪的Key
Set selectedKeys = selector.selectedKeys();
Iterator it = selectedKeys.iterator();
while (it.hasNext()) {
     SelectionKey key = (SelectionKey)it.next();
     // ... deal with I/O event ...
}
// 6.handleAccept处理新接的客户端接入，建立链接
SocketChannel channel = svrChannel.accept();
// 7.设置客户端为非阻塞模式
channel.configureBlocking(false);
channel.socket().setReuseAddress(true);
// 8.想Selector注册读监听
SelectionKey key = socketChannel.register( selector, SelectionKey.OP_READ, ioHandler);
// 9.异步读取消息到缓冲区
int  readNumber =  channel.read(receivedBuffer);
// 10.decode请求消息
Object message = null;
while(buffer.hasRemain()) {
    byteBuffer.mark();
    Object message = decode(byteBuffer);
    if (message == null) {
        byteBuffer.reset();
        break;
    }
    messageList.add(message );
}
if (!byteBuffer.hasRemain())
    byteBuffer.clear();
else
    byteBuffer.compact();
if (messageList != null & !messageList.isEmpty()) {
    for(Object messageE : messageList)
        handlerTask(messageE);
}
// 11.将POJO对象encode成ByteBuffer，调用异步write接口，发送给客户端
socketChannel.write(buffer);
```



##### NIO客户端序列图

![nio客户端序列图](pic/nio客户端序列图.png)



```java
// 1.打开SocketChannel
SocketChannel clientChannel = SocketChannel.open();
// 2.设置clientChannel为非阻塞模式，设置TCP参数
clientChannel.configureBlocking(false);
socket.setReuseAddress(true);
socket.setReceiveBufferSize(BUFFER_SIZE);
socket.setSendBufferSize(BUFFER_SIZE);
// 3.异步连接服务端
boolean connected = clientChannel.connect(new InetSocketAddress(“ip”,port));
// 4.判断是否连接成功,成功跳到10，否则跳到5
if(connected){
    clientChannel.register( selector, SelectionKey.OP_READ, ioHandler);
}else{
    clientChannel.register( selector, SelectionKey.OP_CONNECT, ioHandler);
}
// 5.向reactor线程的多路复用器注册时间OP_CONNECT
clientChannel.register( selector, SelectionKey.OP_CONNECT, ioHandler);
// 6.创建select，启动线程
Selector selector = Selector.open();
New Thread(new ReactorTask()).start();
// 7.Selector轮询准备就绪的Key
Set selectedKeys = selector.selectedKeys();
Iterator it = selectedKeys.iterator();
while (it.hasNext()) {
	if (key.isConnectable())
  	//handlerConnect();
}
// 8.handerConnect
// 9.判断连接是否完成，完成执行10
if (channel.finishConnect())
    registerRead();
// 10.向多路复用器注册读事件
clientChannel.register(selector, SelectionKey.OP_READ, ioHandler);
// 11.异步读请求读取数据到ByteBuffer
int  readNumber =  channel.read(receivedBuffer);
// 12.decode请求消息
Object message = null;
while(buffer.hasRemain()) {
    byteBuffer.mark();
    Object message = decode(byteBuffer);
    if (message == null) {
        byteBuffer.reset();
        break;
    }
    messageList.add(message );
}
if (!byteBuffer.hasRemain())
    byteBuffer.clear();
else
    byteBuffer.compact();
if (messageList != null & !messageList.isEmpty()) {
    for(Object messageE : messageList)
        handlerTask(messageE);
}
// 13.异步写ByteBuffer到socketChannel
socketChannel.write(buffer);
```



