# Java基础

---

## Java是如何做到一次编译，到处运行的

Java可以通过jvm虚拟机实现“一次编译，到处运行”是因为java代码编译后会生成的.class字节码文件，jvm虚拟机会将其翻译成对应操作系统的机器码。例如Springboot打包后生成的jar包，部署到linux虚拟机之后，无需再次编译，可以直接运行。而部分语言（如c++）编译后生成的是对应操作系统的机器码，在windows系统上生成的是.exe文件，而在linux系统上生成的则是.elf文件，因此c++代码在另一个操作系统上就需要重新编译之后才可以运行

## String StringBuilder StringBuffer的区别

String是一种不可变对象，被final修饰，对String对象的修改操作都会生成一个新的String对象。

StringBuilder和StringBuffer都是可变对象，在修改时会在原对象的基础上进行修改

StringBuilder是线程不安全的，StringBuffer是线程安全的，因此在多线程的场景下，必须使用StringBuffer，在追求高效的场景，且无需保证线程安全的场景，才可以使用StringBuilder

---

# IO

## JAVA BIO

BIO就是传统的java io编程

BIO（Blocking IO）：同步阻塞，一个连接一个线程，即客户端有连接请求时，服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池工具来改善

### BIO 工作机制

![](C:\Users\xgw\AppData\Roaming\marktext\images\2025-12-10-13-14-30-image.png)

客户端和服务端通过Socket连接，连接成功之后，可以通过Socket连接建立的虚拟管道来进行数据的传输

```java
public class Client {

    public static void main(String[] args) {
        try {
            Socket socket = new Socket("127.0.0.1", 9999);
            OutputStream os = socket.getOutputStream();
            // 打印流
            PrintStream ps = new PrintStream(os);
            Scanner sc = new Scanner(System.in);
            while(true){
                System.out.println("请输入：");
                String msg = sc.next();
                ps.println(msg);
                ps.flush();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class Server {

    public static void main(String[] args) {

        try {
            System.out.println("========服务器启动...==========");
            ServerSocket ss = new ServerSocket(9999);
            Socket socket = ss.accept();
            InputStream inputStream = socket.getInputStream();
            // 缓冲字节输入流
            BufferedInputStream bis = new BufferedInputStream(inputStream);
            // 缓冲字符输入流 按行读取
            BufferedReader br = new BufferedReader(new InputStreamReader(bis));
            String msg;
            if((msg = br.readLine()) != null){
                System.out.println(msg);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

在以上的通信中，服务端会一致等待客户端的消息，如果客户端没有消息发送，服务端就会一直进入阻塞状态

同时服务端是按照行获取消息的，所以客户端也必须按照行发送消息，否则服务端将一直阻塞

无论客户端还是服务端的socket，只要有一端宕机，另一端就会直接抛出异常

## BIO接收多个客户端

每次有一个客户端发送连接请求，就创建一个新线程来处理这个客户端的请求，就可以实现BIO模式下接收多个客户端

```java
public class Server {

    public static void main(String[] args) {

        try {
            System.out.println("========服务器启动...==========");
            ServerSocket ss = new ServerSocket(9999);
            while(true){
                Socket socket = ss.accept();
                new ServerThreadReader(socket).start();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class ServerThreadReader extends Thread {

    private Socket socket;

    private ServerThreadReader() {

    }

    public ServerThreadReader(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            // 从socket中获取输入流
            InputStream is = socket.getInputStream();
            // 把输入流转换成缓冲流
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            String msg ;
            if((msg = br.readLine()) != null){
                System.out.println("服务端收到消息:" + msg);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 缺点

1. 每接收到一个Socket，都会创建一个线程，线程的竞争，上下文切换等影响性能

2. 每个线程都会占用栈空间和CPU资源

3. 并不是每个Socket都会进行IO操作，无意义的线程处理

4. 客户端并发访问增加时，服务端将呈现1:1的线程开销，访问量越大，系统将发生线程栈溢出，线程创建失败，最终导致进程宕机或僵死

## 伪异步IO编程

采用线程池和任务队列实现，当客户端接入时，将客户端的Socket封装成一个Task，交给后端线程池进行处理

```java
public class Server {

    public static void main(String[] args) {

        try {
            System.out.println("========服务器启动...==========");
            ServerSocket ss = new ServerSocket(9999);
            HandleSocketServerPool pool = new HandleSocketServerPool(6, 10);
            while(true){
                Socket socket = ss.accept();
                Runnable task = new ServerRunnable(socket);
                pool.execute(task);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ServerRunnable implements Runnable {

    private Socket socket;

    public ServerRunnable(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        // 处理客户端的通信需求
        try {
            // 从socket中获取输入流
            InputStream is = socket.getInputStream();
            // 把输入流转换成缓冲流
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            String msg ;
            if((msg = br.readLine()) != null){
                System.out.println("服务端收到消息:" + msg);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class HandleSocketServerPool {
    private ExecutorService executorService;

    public HandleSocketServerPool(int maxThreadNum,int queueSize){
        this.executorService = new ThreadPoolExecutor(
                3,
                maxThreadNum,
                120,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(queueSize)
        );
    }

    public void execute(Runnable task){
        this.executorService.execute(task);
    }
}
```

### 缺点

1. 伪异步IO采用了线程池实现，因此避免了为每个请求创建一个独立线程造成线程资源耗尽的问题，但由于底层依然采用同步阻塞模型，因此无法从根本上解决问题

2. 如果单个消息处理的很慢，或者服务器线程池中的全部线程都被阻塞，那么后续Socket的IO消息都将在队列中排队，新的Socket请求将被拒绝，客户端会发生大量的连接超时

## BIO实现文件上传功能

```java
public class Client {

    public static void main(String[] args) {
        try(
                InputStream in = new FileInputStream("C:\\Users\\xgw\\Desktop\\shea\\java\\java基础.docx");
                ) {
            Socket socket = new Socket("127.0.0.1", 9999);
            DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
            dos.writeUTF(".docx");
            byte[] data = new byte[1024];
            int len;
            while((len = in.read(data)) > 0){
                dos.write(data, 0, len);
            }
            dos.flush();
            socket.shutdownOutput(); // 通知服务端客户端的数据发送完毕
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class ServerReaderThread extends Thread{

    private Socket socket;

    public ServerReaderThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            DataInputStream dis = new DataInputStream(socket.getInputStream());
            String suffix = dis.readUTF();
            System.out.println("服务端接收到文件类型:" + suffix);
            OutputStream os = new FileOutputStream("C:\\Users\\xgw\\Desktop\\shea\\java\\Server\\" +
                    UUID.randomUUID().toString() + suffix);
            byte[] buffer = new byte[1024];
            int len;
            while ((len = dis.read(buffer)) > 0){
                os.write(buffer, 0, len);
            }
            os.close();
            System.out.println("服务端保存文件成功");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

## BIO模式下的端口转发思想

**端口转发**（Port Forwarding）是一种网络技术，将来自一个网络端口的数据流量转发到另一个网络端口或主机。

```java
public class ServerReaderThread extends Thread{

    private Socket socket;

    public ServerReaderThread(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            // 从socket中获取当前客户端的输入流
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String msg;
            while((msg = br.readLine()) != null) {
                // 服务端接收到客户端消息后，需要推送给当前所有的在线socket
                sendMsg2AllClient(msg);
            }
        } catch (Exception e) {
            System.out.println("当前有客户端下线");
            Server.allSocketsOnline.remove(socket);
        }
    }

    private void sendMsg2AllClient(String msg) {
        for (Socket sk : Server.allSocketsOnline) {
            try {
                PrintStream ps = new PrintStream(sk.getOutputStream());
                ps.println(msg);
                ps.flush();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Java NIO

NIO（Non-blocking IO）支持面向缓冲区的，基于通道的IO操作

非阻塞式读取，使一个线程从某通道发送请求或读取数据时，如果目前没有数据可用，则什么都不会获取，而不是保持线程阻塞 

非阻塞式写，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情

### NIO和BIO

BIO是以流的方式处理数据，而NIO是以块的方式处理数据，块IO的效率比流IO的效率高很多

BIO是基于字节流和字符流进行操作，而NIO基于Channel（通道）和Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中，Selector（选择器）用于监听多个通道的事件，因此单个线程就可以监听多个客户端通道

![](C:\Users\xgw\AppData\Roaming\marktext\images\2025-12-11-17-09-15-image.png)

### NIO三大核心

#### Buffer

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，以便访问该块内存

#### Channel

Java NIO通道类似流，但也有些不同。既可以从通道中获取数据，又可以写数据到通道中，但流的读写通常是单向的，通道可以非阻塞读取和写入通道，通道可以支持读取或写入缓冲区，支持异步读写

#### Selector

Selector是一个Java NIO组件，能够检查一个或多个NIO通道，并确定哪些通道已经准备好进行读取或写入，这样，一个线程可以管理多个Channel，从而管理多个网络连接，提高效率

### Buffer

一个用于特定基本数据类型的容器，所有缓冲区都是Buffer抽象类的子类，NIO中的Buffer主要用于与NIO通道进行交互，从中读取或写入数据

#### 基本属性

**容量(capacity)** ：作为一个内存块，Buffer具有一个固定的大小，容量不能为负，并且创建后不能更改

**限制(limit)** ：表示缓冲区可操作数据的大小（limit后的数据不能进行读写），缓冲区的限制不能为负，且不能大于其容量。**写入模式：限制等于buffer的容量，读取模式：limit等于写入的数据量**

**位置(position)** ：下一个要读取或写入数据的索引，缓冲区位置不能为负，且不能大于其限制

**标记(mark)与重置(reset)** ：标记是一个索引，通过Buffer中的mark()方法指定Buffer中一个特定的位置，之后可以通过reset()方法恢复到这个位置

**0<=mark<=position<=limit<=capacity**

### 直接与非直接缓冲区

ByteBuffer可以是两种类型，一种是基于直接内存（非堆内存），另一种是非直接内存（堆内存）。对于直接内存来说，JVM将会在IO操作上有更高的性能，因为它直接作用于本地系统的IO操作，而非直接内存，也就是堆内存中的数据，如果要做IO操作，会先从本进程内存复制到直接内存，再利用本地IO处理

从数据流的角度，非直接内存是下面的作用链

`本地IO --> 直接内存 --> 非直接内存 --> 直接内存 --> 本地IO`

而直接内存是

`本地IO --> 直接内存 --> 本地IO`

因此在做IO处理时，直接内存有更高的效率，直接内存使用allocateDirect创建，但是它比申请普通堆内存需要耗费更高的性能，不过这部分数据是JVM之外的，不会占用应用程序的内存，所以如果有很大的数据要缓存，并且它的生命周期很长，就适合使用**直接内存**

### Channel

Channel表示IO源与目标打开的连接，Channel类似于流，但是Channel不能直接访问数据，只能通过和Buffer交互

#### 通道和流的区别

1. 通道可以同时进行读写，而传统的流只能读或者写

2. 通道可以实现异步读写数据

3. 通道可以从缓冲中读数据，也可以写数据到缓冲中

向文件中写数据

```java
public class ChannelTest {

    @Test
    public void write(){
        try{
            FileOutputStream out = new FileOutputStream("data.txt");
            FileChannel fc = out.getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("hello world".getBytes());
            buffer.flip();
            fc.write(buffer);
        }catch (FileNotFoundException e){
            e.printStackTrace();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

向文件中读数据

```java
@Test
    public void read(){
        try {
            FileInputStream in = new FileInputStream("data.txt");
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            FileChannel fc = in.getChannel();
            fc.read(buffer);
            buffer.flip(); // 切换缓冲区模式，防止读取到空字节
            String s = new String(buffer.array(),0,buffer.remaining());
            System.out.println(s);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

复制文件

```java
@Test
    public void copy(){
        File src = new File("C:\\Users\\xgw\\Pictures\\Screenshots\\屏幕截图 2025-06-18 002903.png");
        File dest = new File("1.png");
        try {
            FileInputStream fis = new FileInputStream(src);
            FileOutputStream fos = new FileOutputStream(dest);
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            FileChannel is = fis.getChannel();
            FileChannel os = fos.getChannel();
            while(true){
                // 必须要先清空缓冲区，然后再写入数据到缓冲区
                buffer.clear();
                // 开始获取一次数据
                int flag = is.read(buffer);
                if(flag == -1){
                    break;
                }
                buffer.flip();
                os.write(buffer);
            }
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

聚集和分散

```java
    @Test
    public void test(){
        try{
            FileInputStream fis = new FileInputStream("data.txt");
            FileChannel is = fis.getChannel();
            FileOutputStream fos = new FileOutputStream("data1.txt");
            FileChannel os = fos.getChannel();
            ByteBuffer buffer1 = ByteBuffer.allocate(4);
            ByteBuffer buffer2 = ByteBuffer.allocate(1024);
            ByteBuffer[] buffers = {buffer1,buffer2};
            is.read(buffers);
            for(ByteBuffer buffer : buffers){
                buffer.flip();
                System.out.print(new String(buffer.array(),0,buffer.remaining()));
            }
            os.write(buffers);
            is.close();
            os.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

聚集和分散，通过读写多个ByteBuffer，一次性处理多个缓冲区，减少上下文的切换，从而提高性能。在处理结构化数据时，也会更加清晰，代码简洁

### Selector

Selector是SelectableChannel对象的多路复用器，Selector可以同时监控多个SelectableChannel的IO状况，也就是说，可以利用Selector单独的一个线程管理多个Channel，Selector是非阻塞式IO的核心

**NIO非阻塞机制实现**

```java
public class Client {
    public static void main(String[] args) {
        try {
            SocketChannel channel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9999));
            channel.configureBlocking(false);
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 发送数据给服务端
            Scanner sc = new Scanner(System.in);
            while(true){
                System.out.println("请输入：");
                String msg = sc.nextLine();
                buffer.put(("Shea：" + msg).getBytes());
                buffer.flip();
                channel.write(buffer);
                buffer.clear();
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class Server {
    public static void main(String[] args) {
        try {
            System.out.println("=========服务器启动==========");
            ServerSocketChannel ssc = ServerSocketChannel.open();
            ssc.configureBlocking(false);
            ssc.bind(new InetSocketAddress(9999));
            Selector selector = Selector.open();
            ssc.register(selector, SelectionKey.OP_ACCEPT);
            // 使用Selector轮询已经就绪好的事件
            while(selector.select() > 0) {
                System.out.println("==========开始处理事件============");
                // 获取选择器中的所有注册的通道中已经就绪好的事件
                Iterator<SelectionKey> selectionKeys = selector.selectedKeys().iterator();
                while(selectionKeys.hasNext()) {
                    SelectionKey next = selectionKeys.next();
                    // 判断事件类型
                    if(next.isAcceptable()) {
                        // 直接获取当前接入的客户端通道
                        ServerSocketChannel channel =(ServerSocketChannel) next.channel();
                        SocketChannel socketChannel = channel.accept();
                        // 切换成非阻塞式通道
                        socketChannel.configureBlocking(false);
                        socketChannel.register(selector, SelectionKey.OP_READ);
                    }else if(next.isReadable()) {
                        // 获取当前选择器上的读就绪事件
                        SocketChannel sk = (SocketChannel) next.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int len = 0;
                        while((len = sk.read(buffer)) > 0){
                            buffer.flip();
                            System.out.println(new String(buffer.array(),0,len));
                            buffer.clear();
                        }
                    }
                    selectionKeys.remove();
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
---
# 设计模式
## 创建者模式
创建者模式主要关注“怎么创建对象”，主要特点是将对象的创建与使用分离，可以降低耦合度，使用者不需要关注如何创建对象，只需要使用即可
### 单例模式
单例模式指的是一个单一的类，该类负责创建自己的实例对象，并且只有这个类可以创建实例对象，且创建的实例对象有且只有一个，该类实例对象可以通过访问方法直接访问，不需要实例化类
**饿汉式**：类加载时就会创建实例对象
```java
public class Singleton {  
	// 私有化构造方法，防止外界创建对象
    private Singleton() {}  
    private static final Singleton instance = new Singleton();  
      
    public static Singleton getInstance() {  
        return instance;  
    }  
}
```
**静态内部类**
JVM在加载外部类过程中，不会加载静态内部类，只有在调用内部类的属性或方法时才会被加载，而静态内部类可以保证类制备实例化一次，并且严格保证实例化顺序
```java
public class Singleton {  
  
    private Singleton() {}  
  
    private static class Holder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    public static Singleton getInstance() {  
        return Holder.INSTANCE;  
    }  
}
```
**枚举类**
枚举类实现单例模式是十分推荐的方法，因为枚举类型是线程安全的，并且只会被装载一次
```java
public enum Singleton {
	INSTANCE;
}
```
**懒汉式**：类加载时不会创建实例对象，只有首次使用的时候才会创建实例对象
```java
public class Singleton {  
  
    private Singleton() {}  
  
    private static Singleton instance;  
  
    public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}
```
但是上述代码是线程不安全的，如果同时有两个线程进入该方法，都会判断instance为null，因此两个线程都会实例化一个对象
线程安全版(double check)
```java
public class Singleton {  
  
    private Singleton() {}  
  
    private static volatile Singleton instance;  
  
    public static Singleton getInstance() {  
        if (instance == null) {  
            synchronized (Singleton.class) {  
                if (instance == null) {  
                    instance = new Singleton();  
                }  
            }  
        }  
        return instance;  
    }  
}
```
**tips**：为什么需要两次判断instance是否为null？为什么要使用volatile关键字？
1. 多线程场景下，如果instance已经被实例化了，则可以不用创建对象，直接返回实例对象。第二次判断instance是否为null，是为了确保只创建了一个实例对象
2. volatile关键字可以确保创建对象过程不被JVM指令重排优化，实例化对象不是一个原子操作，实际上包含了三步，**分配内存空间，初始化对象，将引用指向内存地址**，由于JVM的指令重排，可能会导致其他线程看到未完全初始化的对象，从而出现空指针问题
#### 破坏单例模式
除了枚举类的创建单例对象以外，其他方法的单例模式都可以通过序列化和反射来破坏单例模式
**序列化和反序列化**
```java
public class Main {  
  
    public static void main(String[] args) throws Exception {  
        writeObject();  
        Singleton singleton1 = readObject();  
        Singleton singleton2 = readObject();  
        System.out.println(singleton1 == singleton2);  
    }  
  
    public static Singleton readObject() throws Exception {  
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("C:\\Users\\xgw\\Desktop\\shea\\a.txt"));  
        Singleton singleton = (Singleton) ois.readObject();  
        ois.close();  
        return singleton;  
    }  
  
  
    public static void writeObject() throws IOException {  
        Singleton singleton = Singleton.getInstance();  
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("C:\\Users\\xgw\\Desktop\\shea\\a.txt"));  
        oos.writeObject(singleton);  
        oos.close();  
    }  
}
```
![](assets/Java基础/file-20260203143646169.png)
显而易见，最后得到的两个对象的地址不同，单例模式被破坏
**解决方法**
在Singleton类中重写readResolve方法，直接返回单例对象，在调用readObject方法的时候，会判断是否有重写readResolve方法，如果没有，则会直接new一个新的对象，如果有，则会调用重写的readResolve方法
```java
public Object readResolve() {  
    return Holder.INSTANCE;  
}
```
![](assets/Java基础/file-20260203144610397.png)
**反射**
```java
public class Main {  
  
    public static void main(String[] args) throws Exception {  
        Class<Singleton> clazz = Singleton.class;  
        Constructor<Singleton> constructor = clazz.getDeclaredConstructor();  
        constructor.setAccessible(true);  
        Singleton singleton1 = constructor.newInstance();  
        Singleton singleton2 = constructor.newInstance();  
        System.out.println(singleton1 == singleton2);  
    }  
}
```
![](assets/Java基础/file-20260203143950299.png)
可以看到，反射也可以破坏单例模式
### 工厂模式
定义一个用于创建对象的接口，让子类决定实例化哪个对象，工厂方法使对象的实例化延迟到其工厂的字类
首先需要定义一个工厂接口
```java
public interface CoffeeFactory {  
    Coffee createCoffee();  
}
```
分别实现工厂接口
```java
public class AmericanCoffeeFactory implements CoffeeFactory {  
    @Override  
    public Coffee createCoffee() {  
        return new AmericanCoffee();  
    }  
}
```
```java
public class LatteCoffeeFactory implements CoffeeFactory {  
    @Override  
    public Coffee createCoffee() {  
        return new LatteCoffee();  
    }  
}
```
```java
public interface Coffee {  
}
```
```java
public class AmericanCoffee implements Coffee {  
}
```
```java
public class LatteCoffee implements Coffee {  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        CoffeeStore coffeeStore = new CoffeeStore();  
        coffeeStore.setCoffeeFactory(new LatteCoffeeFactory());  
        Coffee latte = coffeeStore.order();  
        coffeeStore.setCoffeeFactory(new AmericanCoffeeFactory());  
        Coffee american = coffeeStore.order();  
        System.out.println(latte);  
        System.out.println(american);  
    }  
}
```
![](assets/Java基础/file-20260203161819699.png)
我们通过工厂模式将创建对象这一步骤交给工厂来实现，对于不同的实现类，可以提供使用同样的方法来创建对象
**优点**：
1. 将对象的创建与使用分离
2. 添加新产品的时候只需要新增工厂类，无需修改已有代码
3. 每个工厂类只负责创建一种产品
4. 创建逻辑集中在工厂类中
---
**缺点**：
1. 类的数量增加，每次新增产品都需要新增多个类
2. 增加了抽象层和理解难度
### 抽象工厂模式
抽象工厂模式是一种创建型设计模式，它能创建一系列相关的对象，而无需指定其具体类。它提供了一个接口，用于创建**相关或依赖对象的家族**，而不需要明确指定具体类。
在原有咖啡代码不变的情况下，新增以下代码
```java
public abstract class Dessert {    
    public abstract void show();  
}
public class Tiramisu extends Dessert{  
    @Override  
    public void show() {  
        System.out.println("提拉米苏");  
    }  
}
public class MatchaMousse extends Dessert {  
  
    @Override  
    public void show() {  
        System.out.println("抹茶慕斯");  
    }  
}
```
```java
public interface DessertFactory {  
  
    Coffee createCoffee();  
  
    Dessert createDessert();  
}
public class AmericanDessertFactory implements DessertFactory {  
    @Override  
    public Coffee createCoffee() {  
        return new AmericanCoffee();  
    }  
  
    @Override  
    public Dessert createDessert() {  
        return new MatchaMousse();  
    }  
}
public class ItalyDessertFactory implements DessertFactory {  
    @Override  
    public Coffee createCoffee() {  
        return new LatteCoffee();  
    }  
  
    @Override  
    public Dessert createDessert() {  
        return new Tiramisu();  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        ItalyDessertFactory italyDessertFactory = new ItalyDessertFactory();  
        Coffee latte = italyDessertFactory.createCoffee();  
        Dessert tiramisu = italyDessertFactory.createDessert();  
        System.out.println(latte);  
        System.out.println(tiramisu);  
    }  
}
```
![](assets/Java基础/file-20260203200506092.png)
可以看到，我们创建对象时，完全不需要关注其具体如何创建，只需要确定需要创建的是哪一类对象，然后new出对象工厂，就可以根据工厂提供的方法，创建出一类对象
### 原型模式
用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型对象相同的新对象
原型类必须要实现clone()方法
克隆分为浅克隆和深克隆
**浅克隆**：创建一个新的对象，这个对象的属性和原来的对象完全相同，对于非基本类型，属性的引用仍指向原对象的内存地址
**深克隆**：创建一个新的对象，对于非基本类型，也会创建新的对象，其引用会指向新的内存地址
```java
public class User implements Cloneable {  
    @Override  
    public User clone() {  
        try {  
            System.out.println("复制User对象");  
            return (User) super.clone();  
        } catch (CloneNotSupportedException e) {  
            throw new AssertionError();  
        }  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        User user = new User();  
        User user1 = user.clone();  
        System.out.println(user == user1);  
    }  
}
```
![](assets/Java基础/file-20260203210054064.png)
可以看到，通过clone()方法克隆出了一个新对象
#### 深克隆
深克隆有两种方法可以实现，一是字类实现克隆接口，父类的拷贝构造函数中对其进行克隆，二是通过序列化的方式，对类进行深克隆
```java
public class Student implements Cloneable {  
  
    private String name;  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    @Override  
    public Student clone() {  
        try {  
            return (Student) super.clone();  
        } catch (CloneNotSupportedException e) {  
            throw new AssertionError();  
        }  
    }  
}
public class Classroom implements Cloneable{  
  
    private Student student;  
  
    public Student getStudent() {  
        return student;  
    }  
  
    public void setStudent(Student student) {  
        this.student = student;  
    }  
  
    @Override  
    public Classroom clone() {  
        try {  
            Classroom cloned = (Classroom) super.clone();  
            if(this.student != null){  
                this.student = this.student.clone();  
            }  
            return cloned;  
        } catch (CloneNotSupportedException e) {  
            throw new AssertionError();  
        }  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Classroom classroom = new Classroom();  
        Student stu = new Student();  
        stu.setName("Shea");  
        classroom.setStudent(stu);  
        Classroom clone = classroom.clone();  
        clone.getStudent().setName("Shea11");  
        System.out.println("原对象:" + classroom.getStudent().getName());  
        System.out.println("克隆对象:" + clone.getStudent().getName());  
    }  
}
```
![](assets/Java基础/file-20260204143422681.png)
**序列化实现深克隆**
```java
public class Student implements Serializable {  
  
    private String name;  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
}
public class Classroom implements Cloneable, Serializable {  
  
    private Student student;  
  
    public Student getStudent() {  
        return student;  
    }  
  
    public void setStudent(Student student) {  
        this.student = student;  
    }  
  
    @Override  
    public Classroom clone() {  
        try {  
            return (Classroom) super.clone();  
        } catch (CloneNotSupportedException e) {  
            throw new AssertionError();  
        }  
    }  
}

```
```java
public class Main {  
  
    public static void main(String[] args) throws Exception {  
        Classroom classroom = new Classroom();  
        Student stu = new Student();  
        stu.setName("Shea");  
        classroom.setStudent(stu);  
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("C:\\Users\\xgw\\Desktop\\shea\\a.txt"));  
        oos.writeObject(classroom);  
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("C:\\Users\\xgw\\Desktop\\shea\\a.txt"));  
        Classroom clone = (Classroom) ois.readObject();  
        ois.close();  
        oos.close();  
        clone.getStudent().setName("Shea11");  
        System.out.println("原对象:" + classroom.getStudent().getName());  
        System.out.println("克隆对象:" + clone.getStudent().getName());  
    }  
}
```
![](assets/Java基础/file-20260204143952777.png)
### 建造者模式
建造者模式将复杂的对象的构造分离出来，由builder对象专门负责构建其内部的属性，用户不需要知道其内部具体的构造细节
```java
public class Computer {  
  
    private String cpu;  
    private String ram;  
    private String disk;  
    private String keyboard;  
    private String mouse;  
  
    public static class Builder {  
        private String cpu;  
        private String ram;  
        private String disk;  
        private String keyboard;  
        private String mouse;  
  
        public Builder cpu(String cpu) {  
            this.cpu = cpu;  
            return this;  
        }  
        public Builder ram(String ram) {  
            this.ram = ram;  
            return this;  
        }  
        public Builder disk(String disk) {  
            this.disk = disk;  
            return this;  
        }  
        public Builder keyboard(String keyboard) {  
            this.keyboard = keyboard;  
            return this;  
        }  
        public Builder mouse(String mouse) {  
            this.mouse = mouse;  
            return this;  
        }  
        public Computer build() {  
            Computer computer = new Computer();  
            computer.cpu = this.cpu;  
            computer.ram = this.ram;  
            computer.disk = this.disk;  
            computer.keyboard = this.keyboard;  
            computer.mouse = this.mouse;  
            return computer;  
        }  
    }  
  
    @Override  
    public String toString() {  
        return "Computer{" +  
                "cpu='" + cpu + '\'' +  
                ", ram='" + ram + '\'' +  
                ", disk='" + disk + '\'' +  
                ", keyboard='" + keyboard + '\'' +  
                ", mouse='" + mouse + '\'' +  
                '}';  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Computer computer = new Computer.Builder()  
                .cpu("Intel i9")  
                .ram("16GB")  
                .disk("512GB")  
                .keyboard("Logitech")  
                .mouse("Logitech")  
                .build();  
        System.out.println(computer);  
    }  
}
```
![](assets/Java基础/file-20260204144610872.png)
## 结构型模式
结构型模式描述如何将类或对象按某种布局组成更大的结构，它分为类结构型模式和对象结构型模式。前者采用继承机制来组织接口和类，后者采用组合或聚合来组合对象。结构型模式的核心目标是简化系统的设计，通过识别对象之间简单的关系来满足功能需求。
### 代理模式
代理模式，即给某一个对象提供一个代理对象，由代理对象来控制对原对象的操作，代理对象指向原对象的引用
同时，代理模式都可以在不修改原代码的基础上，对代码进行增强
#### 静态代理
静态代理是代理模式的一种，通过创建一个代理类来代表原对象，在编译时就已经确定了代理关系
```java
public interface UserService {    
    void sayHello();  
}
public class UserServiceImpl implements UserService {  
    @Override  
    public void sayHello() {  
        System.out.println("Hello World");  
    }  
}
```
```java
public class UserProxy implements UserService {  
  
    private UserService userService;  
  
    public UserProxy(UserService userService) {  
        this.userService = userService;  
    }  
    @Override  
    public void sayHello() {  
        System.out.println("开始代理，记录日志 " + System.currentTimeMillis());  
        userService.sayHello();  
        System.out.println("代理结束，记录日志 " + System.currentTimeMillis());  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        UserProxy proxy = new UserProxy(new UserServiceImpl());  
        proxy.sayHello();  
    }  
}
```
![](assets/Java基础/file-20260204161001233.png)
#### JDK动态代理
Java中提供了一个动态代理类Proxy，Proxy提供了一个创建代理对象的静态方法newProxyInstance()来获取代理对象
```java
public class ProxyFactory {  
  
    private final UserServiceImpl userServiceImpl = new UserServiceImpl();  
      
    public UserService getUserService() {  
        UserService userService = (UserService) Proxy.newProxyInstance(  
                userServiceImpl.getClass().getClassLoader(),  
                userServiceImpl.getClass().getInterfaces(),  
                (proxy, method, args) -> {  
                    System.out.println("开始代理，记录日志 " + System.currentTimeMillis());  
                    return method.invoke(userServiceImpl, args);   
                }  
        );  
        return userService;  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        ProxyFactory proxyFactory = new ProxyFactory();  
        UserService userService = proxyFactory.getUserService();  
        userService.sayHello();  
    }  
}
```
![](assets/Java基础/file-20260204162448656.png)
代理对象调用sayHello()方法，本质上就是在调用InvocationHandler接口中实现的invoke方法
#### CGLIB动态代理
从newProxyInstance方法的参数中我们可以知道，jdk动态代理的代理类必须是实现了某个接口的类，那么对于没有实现接口的类，我们可以使用cglib动态代理来实现
```java
public class UserServiceImpl  {  
    public void sayHello() {  
        System.out.println("Hello World");  
    }  
}
public class ProxyFactory implements MethodInterceptor {  
  
    private final UserServiceImpl target = new UserServiceImpl();  
  
    public UserServiceImpl getProxyObject(){  
        Enhancer enhancer = new Enhancer();  
        // 设置父类的字节码对象  
        enhancer.setSuperclass(UserServiceImpl.class);  
        // 设置回调函数  
        enhancer.setCallback(this);  
        // 创建代理对象  
        UserServiceImpl userService = (UserServiceImpl) enhancer.create();  
        return userService;  
    }  
  
    @Override  
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {  
        System.out.println("打印日志，当前时间："+ System.currentTimeMillis());  
        return method.invoke(target, args);  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        ProxyFactory proxyFactory = new ProxyFactory();  
        UserServiceImpl proxyObject = proxyFactory.getProxyObject();  
        proxyObject.sayHello();  
    }  
}
```
![](assets/Java基础/file-20260204194815176.png)
**cglib动态代理 VS jdk动态代理**
1. jdk动态代理的代理类必须要实现接口，cglib动态代理的代理类则不需要，cglib是通过继承目标类创建字类作为代理类，因为cglib是通过继承创建代理类，因此对于final修饰的方法，cglib无法进行代理
2. jdk代理比cglib代理性能更差，因为jdk代理使用的是反射机制，而cglib使用的是字节码
**优点**：
- 代理模式在客户端与目标对象之间起到了一个中介作用和保护目标对象的作用
- 代理对象可以扩展目标对象的功能
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度
**缺点**：
- 增加了系统的复杂度
### 适配器模式
**将一个类的接口转换成客户端期望的另一个接口**，使得原本由于接口不兼容而不能一起工作的那些类能一起工作，类似于电源适配器的作用
适配器模式分为类适配器模式和对象适配器模式
#### 类适配器模式
```java
public interface TypeC {    
    void connectTypeC();  
}
public class TypeCImpl implements TypeC{  
    @Override  
    public void connectTypeC() {  
        System.out.println("TypeC connected");  
    }  
}
```
```java
public interface USB {  
    void connectUSB();  
}
public class USBImpl implements USB{  
    @Override  
    public void connectUSB() {  
        System.out.println("USB connected");  
    }  
}
```
```java
public class Computer {  
    public void connectAdapter(USB usb){  
        usb.connectUSB();  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Computer computer = new Computer();  
        USB usb = new USBImpl();  
        computer.connectAdapter(usb);  
        TypeC2USBAdapter adapter = new TypeC2USBAdapter();  
        computer.connectAdapter(adapter);  
    }  
}
```
![](assets/Java基础/file-20260204202237702.png)
上述代码中，Computer类原先只能连接USB接口，在适配器的作用下，将TypeC接口转换成USB接口，成功让Computer类连接上TypeC接口
#### 对象适配器模式
对象适配器模式可采用将现有组件库中已经实现的组件引入适配器类中，该类同时实现当前系统的业务接口
```java
public class TypeC2USBAdapter implements USB {  
    private TypeC typeC;  
      
    public TypeC2USBAdapter(TypeC typeC) {  
        this.typeC = typeC;  
    }  
    @Override  
    public void connectUSB() {  
        System.out.println("适配器将TypeC转换为USB");  
        typeC.connectTypeC();  
    }  
}
```
只需要将原先的继承的类改为成员变量形式即可
### 装饰者模式
装饰器模式，即在不改变现有对象结构的情况下，动态地给该对象增加一些职责
```java
public interface Coffee {  
  
    Double getPrice();  
  
    String getDesc();  
}
public class AmericanCoffee implements Coffee {  
    @Override  
    public Double getPrice() {  
        return 9.9;  
    }  
  
    @Override  
    public String getDesc() {  
        return "American Coffee";  
    }  
}
```
```java
abstract class CoffeeDecorator implements Coffee {  
  
    protected Coffee coffee;  
  
    public CoffeeDecorator(Coffee coffee) {  
        this.coffee = coffee;  
    }  
  
    @Override  
    public Double getPrice() {  
        return coffee.getPrice();  
    }  
  
    @Override  
    public String getDesc() {  
        return coffee.getDesc();  
    }  
}
public class MilkDecorator extends CoffeeDecorator {  
  
    public MilkDecorator(Coffee coffee) {  
        super(coffee);  
    }  
  
    @Override  
    public Double getPrice() {  
        return super.getPrice() + 2.0;  
    }  
  
    @Override  
    public String getDesc() {  
        return super.getDesc() + ", Milk";  
    }  
}
public class SugarDecorator extends CoffeeDecorator {  
  
    public SugarDecorator(Coffee coffee) {  
        super(coffee);  
    }  
  
    @Override  
    public Double getPrice() {  
        return super.getPrice() + 1.0;  
    }  
  
    @Override  
    public String getDesc() {  
        return super.getDesc() + ", Sugar";  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Coffee coffee = new AmericanCoffee();  
        System.out.println(coffee.getDesc() + " $" + coffee.getPrice());  
  
        coffee = new MilkDecorator(coffee);  
        System.out.println(coffee.getDesc() + " $" + coffee.getPrice());  
  
        coffee = new SugarDecorator(coffee);  
        System.out.println(coffee.getDesc() + " $" + coffee.getPrice());  
    }  
}
```
上述代码中，我们可以在不修改原有的咖啡类的同时，对咖啡进行加糖和加牛奶的操作
**静态代理和装饰者模式的区别**
相同点：
-  都要实现与目标类相同的业务接口
*  在两个类中都要声明目标对象
-  都可以在不修改目标类的前提下增强目标方法
不同点：
- 目的不同 装饰者是为了增强目标对象 静态代理是为了保护和隐藏目标对象
- 获取目标对象构建的地方不同 装饰者是由外界传递进来，可以通过构造方法传递 
- 静态代理是在代理类内部创建，以此来隐藏目标对象
### 桥接模式
将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现，从而降低了抽象和实现这两个可变维度的耦合度
```java
public interface Color {  
    void show();  
}
public class Green implements Color{  
    @Override  
    public void show() {  
        System.out.print("Green ");  
    }  
}
public class Blue implements Color{  
    @Override  
    public void show() {  
        System.out.print("Blue ");  
    }  
}
```
```java
public abstract class Shape {  
  
    protected Color color;  
  
    public Shape(Color color) {  
        this.color = color;  
    }  
  
    abstract void draw();  
}
public class Circle extends Shape {  
  
    public Circle(Color color) {  
        super(color);  
    }  
  
    @Override  
    void draw() {  
        color.show();  
        System.out.println("Circle");  
    }  
}
public class Tangle extends Shape {  
  
    public Tangle(Color color) {  
        super(color);  
    }  
  
    @Override  
    void draw() {  
        color.show();  
        System.out.println("Tangle");  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Shape circle = new Circle(new Blue());  
        Shape circle2 = new Circle(new Green());  
        Shape tangle = new Tangle(new Blue());  
        Shape tangle2 = new Tangle(new Green());  
        circle.draw();  
        circle2.draw();  
        tangle.draw();  
        tangle2.draw();  
    }  
}
```
![](assets/Java基础/file-20260205104429381.png)
可以看到，如果我们不使用桥接模式来实现，可能会导致类爆炸，每一种颜色的图形就需要一个单独的类，而使用桥接模式，可以通过聚合来减少类的增加。如果有新的颜色或者图形增加，只需要新增一个类实现接口即可
### 外观模式
又叫门面模式，是一种通过多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式，该模式对外有一个统一接口，外部应用程序不关心内部子系统的具体实现细节
```java
public class TV {  
    public void on(){  
        System.out.println("TV is on");  
    }  
  
    public void off(){  
        System.out.println("TV is off");  
    }  
}
public class AirCondition {  
    public void on(){  
        System.out.println("AirCondition is on");  
    }  
  
    public void off(){  
        System.out.println("AirCondition is off");  
    }  
}
public class Light {  
    public void on(){  
        System.out.println("Light is on");  
    }  
  
    public void off(){  
        System.out.println("Light is off");  
    }  
}
```
```java
public class SmartAssistFacade {  
  
    private final Light light;  
    private final TV tv;  
    private final AirCondition airCondition;  
  
    public SmartAssistFacade() {  
        this.light = new Light();  
        this.tv = new TV();  
        this.airCondition = new AirCondition();  
    }  
  
    public void on(){  
        light.on();  
        tv.on();  
        airCondition.on();  
    }  
  
    public void off(){  
        light.off();  
        tv.off();  
        airCondition.off();  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        SmartAssistFacade smartAssistFacade = new SmartAssistFacade();  
        smartAssistFacade.on();  
        smartAssistFacade.off();  
    }  
}
```
![](assets/Java基础/file-20260205182929427.png)
原先要控制多个家具的开关，我们需要对每一个家具都调用on和off方法，但是通过外观模式，创建一个类统一来开关家具
**优点**：
- 降低了子系统和客户端的耦合度，使得子系统的变化不会影响调用它的客户类
- 对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更容易
**缺点**：
- 不符合开闭原则，修改麻烦
### 组合模式
允许你将对象组合成树形结构来表示“部分-整体”的层次关系。组合模式让客户端可以统一地处理单个对象和组合对象，无需关心它们的具体差异
```java
public interface PopulationNode {  
    int computePopulation();  
}
public class Province implements PopulationNode {  
  
    private final String name;  
    private List<PopulationNode> cities =  new ArrayList<>();  
  
    public Province(String name) {  
        this.name = name;  
    }  
  
    @Override  
    public int computePopulation() {  
        return cities.stream().mapToInt(PopulationNode::computePopulation).sum();  
    }  
}
public class City implements PopulationNode {  
  
    private final String name;  
    List<PopulationNode> districts = new ArrayList<>();  
  
    public City(String name) {  
        this.name = name;  
    }  
  
    public void addDistrict(District district) {  
        districts.add(district);  
    }  
  
    @Override  
    public int computePopulation() {  
        return districts.stream().mapToInt(PopulationNode::computePopulation).sum();  
    }  
}
public class District implements PopulationNode{  
  
    private final String name;  
    private final int population;  
  
    public District(String name, int population) {  
        this.name = name;  
        this.population = population;  
    }  
  
    @Override  
    public int computePopulation() {  
        return population;  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Province province = new Province("江西");  
        City city1 = new City("南昌");  
        City city2 = new City("景德镇");  
        District district1 = new District("东湖区", 100000);  
        District district2 = new District("昌江区", 200000);  
        District district3 = new District("珠山区", 300000);  
        District district4 = new District("abc", 400000);  
        city1.addDistrict(district1);  
        city1.addDistrict(district2);  
        city2.addDistrict(district3);  
        city2.addDistrict(district4);  
        province.addCity(city1);  
        province.addCity(city2);  
        System.out.println(province.computePopulation());  
    }  
}
```
### 享元模式
通过共享技术来有效地支持大量细粒度对象的复用。它通过共享已经存在的对象来大幅度减少需要创建对象的数量，避免创建大量相似对象，从而提高系统资源利用率
享元模式将对象的内部状态和外部状态分离
- **内部状态**：对象中不变的，可以共享的部分
- **外部状态**：对象中变化的，不能共享的部分，有客户端代码在需要时传入
```java
public abstract class AbstractBox {  
  
    // 获取图形  
    public abstract String getShape();  
    // 显示图形及颜色  
    public void display(String color){  
        System.out.println("Shape: " + getShape() + ", Color: " + color);  
    }  
}
public class IBox extends AbstractBox {  
    @Override  
    public String getShape() {  
        return "I";  
    }  
}
public class LBox extends AbstractBox {  
    @Override  
    public String getShape() {  
        return "L";  
    }  
}
public class OBox extends AbstractBox {  
    @Override  
    public String getShape() {  
        return "O";  
    }  
}
```
```java
public class BoxFactory {  
  
    private final HashMap<String,AbstractBox> map;  
    private static final BoxFactory instance = new BoxFactory();  
    private BoxFactory() {  
        this.map = new HashMap<>();  
        map.put("I", new IBox());  
        map.put("O", new OBox());  
        map.put("L", new LBox());  
    }  
  
    public static BoxFactory getInstance() {  
        return instance;  
    }  
  
    public AbstractBox getShape(String name){  
        return map.get(name);  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        BoxFactory instance = BoxFactory.getInstance();  
        AbstractBox i = instance.getShape("I");  
        AbstractBox o = instance.getShape("O");  
        AbstractBox l = instance.getShape("L");  
        i.display("Red");  
        o.display("Blue");  
        l.display("Green");  
        AbstractBox i1 = instance.getShape("I");  
        i1.display("Gray");  
        System.out.println(i == i1);  
    }  
}
```
![](assets/Java基础/file-20260206150211606.png)
两次从工厂中取出来的对象是同一个，没有新建一个对象，而是利用缓存中的原有的对象
**优点**：
- 极大减少内存中相似或相同对象数量，节约系统资源，提高系统性能
- 享元模式中的外部状态相对独立，且不影响内部状态
**缺点**：
- 为了使对象可以共享，需要将享元对象的部分状态外部化，分离内部状态和外部状态，是程序逻辑复杂
## 行为型模式
行为型模式用于描述程序在运行时复杂的流程控制，即描述多个类或对象之间怎么相互协作共同完成单个对象都无法完成的任务
行为型模式分为类行为模式和对象行为模式，前者采用继承机制在类间分派行为，后者采用组合或聚合在对象间分配行为
### 模版方法模式
定义一个操作中的算法骨架，而将算法的一些步骤延迟到子类中，使得子类可以不改变改算法结构的情况下重定义该算法的某些特定步骤
```java
public abstract class Vegetables {  
  
    public final void cook(){  
        pourOil();  
        headOil();  
        pourVegetable();  
        pourSauce();  
        fry();  
    }  
  
    public void pourOil(){  
        System.out.println("倒油");  
    }  
    public void headOil(){  
        System.out.println("加热");  
    }  
    public abstract void pourVegetable();  
    public abstract void pourSauce();  
  
    public void fry(){  
        System.out.println("翻炒");  
    }  
}
public class Cabbage extends Vegetables{  
    @Override  
    public void pourVegetable() {  
        System.out.println("倒入包菜");  
    }  
  
    @Override  
    public void pourSauce() {  
        System.out.println("加入辣椒");  
    }  
}
public class ChineseCabbage extends Vegetables {  
    @Override  
    public void pourVegetable() {  
        System.out.println("倒入白菜");  
    }  
  
    @Override  
    public void pourSauce() {  
        System.out.println("加入蒜蓉");  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Cabbage cabbage = new Cabbage();  
        cabbage.cook();  
        System.out.println("================================");  
        ChineseCabbage chineseCabbage = new ChineseCabbage();  
        chineseCabbage.cook();  
    }  
}
```
![](assets/Java基础/file-20260206195809389.png)
模版方法模式中要有一个final修饰的模版方法，不可被子类继承，防止被重写后改变了算法的框架
**优点**：
- 提高了代码的复用性
- 实现了反向控制，通过父类调用子类的操作，通过子类对具体实现扩展不同的行为，实现了反向控制
**缺点**：
- 对每个不同的实现都要定义一个类，导致类数量增肌，系统庞大
- 父类中的抽象方法都由子类实现，子类执行的结果会影响父类的结果，提高了代码阅读的难度
### 策略模式
定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的用户，把使用算法的职责和算法的实现分隔开，并委派给不同的对象对这些算法进行管理
```java
public interface CustomService {  
    String show();  
}
@Component  
@SupportCustom(UserType.NORMAL)  
public class NormalCustomService implements CustomService{  
    @Override  
    public String show() {  
        return "Normal Custom Service";  
    }  
}
@Component  
@SupportCustom(UserType.SMALL_R)  
public class SmallRCustomService implements CustomService {  
    @Override  
    public String show() {  
        return "Small R Custom Service";  
    }  
}
@Component  
@SupportCustom(UserType.BIG_R)  
public class BigRCustomService implements CustomService {  
    @Override  
    public String show() {  
        return "Big R Custom Service";  
    }  
}
@Component  
@SupportCustom(UserType.PERSONAL)  
public class PersonalCustomService implements CustomService {  
    @Override  
    public String show() {  
        return "Personal Custom Service";  
    }  
}
@Component  
public class DefaultCustomService implements CustomService {  
    @Override  
    public String show() {  
        return "Default Custom Service";  
    }  
}
```
### 命令模式
它将一个请求封装为一个对象，从而使你可以用不同的请求对客户进行参数化，支持请求的排队、记录、撤销等操作。
```java
public interface Command {  
    void execute();  
  
    // 可选操作，支持撤销  
    void undo();  
}
public class LightOnCommand implements Command {  
  
    private Light light;  
  
    public LightOnCommand(Light light) {  
        this.light = light;  
    }  
  
    @Override  
    public void execute() {  
        light.turnOn();  
    }  
  
    @Override  
    public void undo() {  
        light.turnOff();  
    }  
}
```
```java
public class Light {  
  
    public void turnOn() {  
        System.out.println("Light is on");  
    }  
  
    public void turnOff() {  
        System.out.println("Light is off");  
    }  
}
```
```java
public class Invoker {  
  
    private Command command;  
  
    public void setCommand(Command command) {  
        this.command = command;  
    }  
  
    public void pressButton(){  
        command.execute();  
    }  
  
    public void pressUndoButton(){  
        command.undo();  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        Light light = new Light();  
        Command lightOn = new LightOnCommand(light);  
  
        Invoker invoker = new Invoker();  
        invoker.setCommand(lightOn);  
  
        invoker.pressButton(); // 执行命令  
        invoker.pressUndoButton(); // 撤销命令  
    }  
}
```
将Command请求封装成对象，从而可以将调用者(Invoker)和接受者(Receiver)解耦
### 责任链模式
为了避免请求发送者与多个请求处理者耦合在一起，让多个对象都有机会处理请求，将这些对象连接成一条链，并沿着这条链传递请求，直到有对象处理它为止
```java
public abstract class Handler {  
    protected Handler successor;  
  
    public void setSuccessor(Handler successor) {  
        this.successor = successor;  
    }  
  
    public abstract void handleRequest(int request);  
}
public class HandlerA extends Handler {  
    @Override  
    public void handleRequest(int request) {  
        if(request <= 10){  
            System.out.println("HandlerA 处理请求" + request);  
        }else if(successor != null){  
            successor.handleRequest(request);  
        }  
    }  
}
public class HandlerB extends Handler {  
    @Override  
    public void handleRequest(int request) {  
        if(request <= 20){  
            System.out.println("HandlerB 处理请求" + request);  
        }else if(successor != null){  
            successor.handleRequest(request);  
        }  
    }  
}
public class HandlerC extends Handler {  
    @Override  
    public void handleRequest(int request) {  
        if(request <= 30){  
            System.out.println("HandlerC 处理请求" + request);  
        } else {  
            System.out.println("无处理请求的方法 " + request);  
        }  
    }  
}
```
```java
public class Main {  
    public static void main(String[] args) {  
        Handler handlerA = new HandlerA();  
        Handler handlerB = new HandlerB();  
        Handler handlerC = new HandlerC();  
        handlerA.setSuccessor(handlerB);  
        handlerB.setSuccessor(handlerC);  
        int[] request = {5,15,25,35};  
        for (int i : request) {  
            handlerA.handleRequest(i);  
        }  
    }  
}
```
![](assets/Java基础/file-20260207161359715.png)
通过责任链模式，我们可以不需要知道具体是哪个对象处理请求，只需要将请求传给第一个对象即可，如果第一个对象无法处理，则会自动传给下一个对象，每个对象都只处理自己职责范围内的请求
### 状态模式
将对象的状态封装成独立的类，并将委托行为委托给当前状态对象，而不是在对象内部使用大量的条件语句（if-else 或 switch-case）来判断状态
```java
public interface VoteState {  
    void vote(String user,String voteItem,VoteManager manager);  
}
public class NormalVoteState implements VoteState {  
    @Override  
    public void vote(String user, String voteItem, VoteManager manager) {  
        manager.getVoteMap().put(user, voteItem);  
        System.out.println("恭喜投票成功");  
    }  
}
public class RepeatVoteState implements VoteState {  
    @Override  
    public void vote(String user, String voteItem, VoteManager manager) {  
        System.out.println("请勿重复投票");  
    }  
}
public class SpiteVoteState implements VoteState {  
    @Override  
    public void vote(String user, String voteItem, VoteManager manager) {  
        String s = manager.getVoteMap().get(user);  
        if(s!=null) {  
            manager.getVoteMap().remove(user);  
        }  
        System.out.println("疑似刷票行为，取消投票资格");  
    }  
}
public class BlackVoteState implements VoteState {  
    @Override  
    public void vote(String user, String voteItem, VoteManager manager) {  
        System.out.println("黑名单用户，没有投票资格");  
    }  
}
```
```java
public class VoteManager {  
    private VoteState state = null;  
    private Map<String,String> voteMap = new HashMap<>();  
    private Map<String,Integer> voteCount = new HashMap<>();  
  
    public Map<String,String> getVoteMap() {  
        return voteMap;  
    }  
  
    public void vote(String user,String voteItem){  
        Integer votes = this.voteCount.get(user);  
        if(votes == null){  
            votes = 0;  
        }  
        votes++;  
        this.voteCount.put(user,votes);  
        if(votes == 1){  
            state = new NormalVoteState();  
        } else if (votes > 1 && votes <= 5) {  
            state = new SpiteVoteState();  
        } else if (votes > 5 && votes <= 8) {  
            state = new SpiteVoteState();  
        } else if (votes > 8){  
            state = new BlackVoteState();  
        }  
        state.vote(user,voteItem,this);  
    }  
}
```
```java
public class Main {  
  
    public static void main(String[] args) {  
        VoteManager voteManager = new VoteManager();  
        for (int i = 0; i < 10; i++) {  
            voteManager.vote("shea","A");  
        }  
    }  
}
```
![](assets/Java基础/file-20260207214800027.png)
