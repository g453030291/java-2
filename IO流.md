#### 一、什么是比特？什么是字节？什么是字符？他们长度是多少？各有什么区别？

Bit最小的二进制单位，是计算机的操作部分，取值0或1。

Byte是计算机操作数据的最小单位由8位bit组成。取值（-128-127）

Char是用户的可读写的最小单位，在Java里面由16位bit组成。取值（0-65535）

#### 二、字节流和字符流的区别？

字节流：操作byte类型的数据，主要操作类是OutputStream、InputStream的子类；不使用缓冲区，直接对文件本身操作。

字符流：操作字符类型数据，主要操作类是Reader、Writer的子类；使用缓冲区操作字符，不关闭流就不会输出任何内容。

#### 三、字节流和字符流之间如何相互转换？

整个IO包实际上分为字节流和字符流，但是出了这两个流之外，还存在一组字节流-字符流的转换类。

OutputStreamWriter：是Writer的子类，将输出的字符流变为字节流，即将一个字符流的输出对象变为字节流输出对象。

InputStreamReader：是Reader的子类，将输入的字节流变为字符流，即将一个字节流的输入对象变为字符流的输入对象。

#### 四、什么是NIO？

NIO与原来的IO有同样的作用和目的，他们之间最重要的区别是数据打包和传输的方式。原来的IO以流的方式处理数据，而NIO以块的方式处理数据。

面向流的IO系统一次一个字节的处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的IO通常相当慢。

一个面向块的IO系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按（流式的）字节数据要快得多。但是面向块的IO缺少一些面向流的IO所具有的优雅性和简单性。

#### 五、什么是AIO？

Java AIO即Async非阻塞，是异步非阻塞的IO。

#### 六、什么是BIO？

Java BIO即Block IO，同步并阻塞的IO。BIO就是传统的java.io包下面的代码实现。

#### 七、Java 7中关闭IO的更优雅的方式是什么？

在Java 7中新定义了一个AutoCloseable接口，他位于java.lang包下。主要在try-with-resources语句中会被自动调用，用于自动释放资源。

try-with-resources语句是JDK1.7中一个新的异常处理机制，更方便简洁的关闭在try-catch语句块中使用的资源。

#### 八、BIO、NIO及AIO三者之间的区别和联系有哪些？

BIO（Blocking IO）：同步阻塞IO模式，数据的读取写入必须阻塞在一个线程内等待其完成。这里假设一个烧开水的场景，有一排水壶在烧开水，BIO的工作模式就是，叫一个线程停留在一个水壶那，直到这个水壶烧开，才去处理下一个水壶。但是实际上线程在等待水壶烧开的时间段什么都没有做。

NIO（New IO）：同时支持阻塞与非阻塞模式，但这里我们以其同步非阻塞IO模式来说明，那么什么叫做非阻塞？如果还拿烧开水来说，NIO的做法是叫一个线程不断的轮询每个水壶的状态，看看是否有水壶的状态发生了改变，从而进行下一步的操作。

AIO（Asynchronous IO）：异步非阻塞IO模型。异步非阻塞与同步非阻塞的区别在哪里？异步非阻塞无需一个线程去轮询所有IO操作的状态改变，在相应的状态改变后，系统会通知对应的线程来处理。对应到烧开水中就是，为每个水壶上面装了一个开关，水烧开之后，水壶会自动通知我水烧开了。

#### 九、使用BIO实现文件的读取和写入。

```java
//initializes The Object
User1 user = new User1();
user.setName("A");
user.setAge(23);
System.out.println(user);

//Write Obj to File
ObjectOutputStream oos = null;
try{
    oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
    oos.writeObject(user);
}catch(IOException e){
    e.printStackTrace();
}finally{
    IOUtils.closeQuietly(oos);
}

//Read Obj from File
File file = new File("tempFIle");
ObjectInputStream ois = null;
try{
    ois = new ObjectInputStream(new FileInputStream(file));
    User1 newUser = (User1)ois.readObject();
    System.out.println(newUser);
}catch(IOException e){
    e.printStackTrace();
}catch(ClassNotFoundException e){
    e.printStackTrace();
}finally{
    IOUtils.closeQuieyly(ois);
    try{
        FileUtils.forceDelete(file);
    }catch(IOException e){
        e.printStackTrace();
    }
}
```

#### 十、使用NIO实现文件的读取和写入。

```java
static void readNIO(){
    String pathname = "/user/aaa/ff.jpg";
    FileInputStream fin = null;
    try{
        fin = new FileInputStream(new File(pathname));
        FileChannel channel = fin.getChannel();
        
        int capacity = 100;//字节
        ByteBuffer bf = ByteBuffer.allocate(capacity);
        System.out.println("限制是:"+bf.limit()+"容量是:"+bf.capacity()+"位置是:"+bf.position());
        int length = -1;
        
        while((length = channel.read(bf))!=-1){
            /*
    		 *注意，读取后，将位置置为0，将limit置为容量，已备下次读入到字节缓冲中，从0开始存储 
             */
            bf.clear();
            byte[] bytes = bf.array();
            System.out.write(bytes,0,length);
            System.out.println();
            System.out.println("限制是:"+bf.limit()+"容量是:"+bf.capacity()+"位置是:"+bf.position());
        }
        channel.close();
    }catch(FileNotFoundException e){
        e.printStackTrace();
    }catch(IOException e){
        e.printStackTrace();
    }finally{
        if(fin!=null){
            try{
                fin.close;
            }catch(IOException e){
                e.printStackTrace();
            }
        }
    }
}
```

```java
static void writeNIO(){
    String filename = "out.txt";
    FileOutputStream fos = null;
    try{
        fos = new FileOutputStream(new File(filename));
        FileChannel channel = fos.getChannel();
        ByteBuffer src = Charset.forName("utf8").encode("nihaonihaonihao");
        //字节缓冲的容量和limit会随着数据长度变化，不是固定不变的
        System.out.println("容量初始化和limit:"+src.capacity()+","+src.limit());
        int length = 0;
        while((length=channel.write(src))!=0){
            /*
             *注意,这里不需要clear,将缓冲中的数据写入到通道中后,第二次接着上一次的顺序往下读
             */
        	System.out.println("写入长度:"+length);
        }
    }catch(FileNotFoundException e){
        e.printStackTrace();
    }catch(IOException e){
        e.printStackTrace();
    }finally{
        if(fos!=null){
            try{
                fos.close();
            }catch(IOException e){
                e.printStackTrace();
            }
        }
    }
}
```

#### 十一、使用AIO实现文件的读取和写入。

```java
public class ReadFromFile{
    public static void main(String[] args)throws Exception{
        Path file = paths.get("/user/a.txt");
        AsynchronousFileChannel channel = AsynchronousFileChannel.open(file);
        
        ByteBuffer buffer = ByteBuffer.allocate(100_000);
        Future<Integer> result = channel.read(buffer,0);
        
        while(!resule.isDone()){
            ProfitCaculator.calculateTax();
        }
        Integer bytesRead = result.get();
        System.out.println("Bytes read["+bytesRead+"]");
    }
}

class ProfitCalculator{
    public ProfitCalculator(){}
    public static void calculateTax(){}
}

public class WriteToFile{
    AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
    	Paths.get("/asynchronous.txt"),StandardOpenOption.READ,
        StandardOpenOption.WRITE,StandardOpenOption,Create);
    CompletionHandler<Integer,Object> handler = new CompletionHandler<Integer,Object>(){
        @Override
        public void failed(Throwable e,Object attachment){
            System.err.println("Attachment:"+attachment+"failed with:");
            e.printStackTrace();
        }
        
        System.out.println("Main Thread ID:"+Thread.currentThread().getId());
        fileChannel.write(ByteBuffer.wrap("Sample".getBytes()),0,"First Write",handler);
        fileChannel.write(ByteBuffer.wrap("Box".getBytes()),0,"Second Write",handler);
    }
}
```

#### 十二、Java中BIO、NIO、AIO分别适用哪些场景？

BIO：方式适用于连接数目较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。

NIO：方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器中，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

AIO：方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

#### 十三、什么是同步？什么是异步？

同步与异步描述的是被调用者。

举例：如果A调用B

同步：B在收到A的调用后，会立即执行要做的事。A的本次调用可以得到结果。

异步：B在收到A的调用后，不保证会立即执行要做的事，但是保证会去做。B在做好了之后会通知A。A的本次调用得不到结果，但是B执行完之后会通知A。

#### 十四、什么是阻塞？什么是非阻塞？

阻塞与非阻塞描述的是调用者。

举例：如果A调用B

阻塞：A在发出调用后，要一直等待，等着B返回结果。

非阻塞：A在发出调用后，不需要等待，可以去做自己的事。

#### 十五、同步、异步、阻塞、非阻塞之间有什么区别？

同步不一定阻塞，异步也不一定不阻塞。没有必然关系。两两之间可以任意组合，同步阻塞、同步非阻塞、异步阻塞、异步非阻塞。

#### 十六、IO模型有哪五种？

阻塞式IO模型

非阻塞IO模型

IO复用模型

信号驱动IO模型

异步IO模型

#### 十七、简单介绍一下阻塞IO模型？

最传统的一种IO模型，即在读写数据过程中会发生阻塞现象。

当用户线程发出IO请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU。当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。

典型的阻塞IO模型的例子为：

data=socket.read();

如果数据没有就绪，就会一直阻塞在read方法。

#### 十八、简单介绍一下非阻塞IO模型？

当用户线程发起一个read操作后，并不需要等待，而是马上就得到一个结果。如果结果是一个error时，他就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。

所以事实上，在非阻塞IO模型中，用户线程需要不断地询问内核数据是否就绪，也就说非阻塞IO不会交出CPU，而会一直占用CPU。

典型的非阻塞IO模型一般如下

```java
while(true){
    data = socket.read();
    if(data!=error){
        //处理数据
        break;
    }
}
```

但是对于非阻塞IO有一个非常严重的问题，在while循环中需要不断的去询问内核数据是否就绪，这样会导致CPU占用率非常高，因此一般情况下很少使用while循环这种方式来读取数据。

#### 十九、简单介绍一下多路复用IO模型？

多路复用IO模型是目前使用的比较多的模型。Java NIO实际上就是多路复用IO。

在多路复用IO模型中，会有一个线程不断去轮训多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。因为在多路复用IO模型中，只需要使用一个线程就可以管理多个socket，系统不需要建立新的进程或线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。

在Java NIO中，是通过selector.select()去查询每个通道是否有到达事件，如果没有事件，则一直阻塞在那里，因此这种方式会导致用户线程的阻塞。

也许有朋友会说，我可以采用多线程+阻塞IO达到类似效果，但是由于在多线程+阻塞IO中，每个socket对应一个线程，这样会造成很大的资源占用，并且尤其是对长连接来说，线程的资源一直不会释放，如果后面陆续有很多连接的话，就会造成性能上的瓶颈。

而多路复用IO模式，通过一个线程就可以管理多个socket，只有当socket真正有读写事件发生才会占用资源来进行实际的读写操作。因此，多路复用IO比较适合连接数比较多的情况。

另外多路复用IO为何比阻塞IO模型的效率高是因为在非阻塞IO中，不断的询问socket状态是通过用户线程去进行的，而在多路复用IO中，轮询每个socket状态是内核在进行的，这个效率要比用户线程搞得多。

不过要注意的是，多路复用IO模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件逐一进行响应。因此对于多路复用IO模型来说，一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询。

#### 二十、简单介绍一下信号驱动IO模型？

在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。

#### 二十一、简单介绍一个什么是异步IO模型？

异步IO模型是比较理想的IO模型，在异步IO模型中，当用户线程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从内核的角度，当它受到一个asynchronous read之后，它会立刻返回，说明read请求已经成功发起了，因此不会对用户线程产生任何block。然后，内核会等待数据准备完成，然后将数据拷贝到用户线程，当这一切都完成之后，内核会给用户线程发送一个信号，告诉它read操作完成了。也就说用户线程完全不需要实际的整个IO操作是如何进行的，只需要先发起一个请求，当接受内核返回的成功信号时表示IO操作已经完成，可以直接去使用数据了。

也就说在异步IO模型中， IO操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户，线程操作已完成。用户线程中不需要再次调用IO函数进行具体的读写。这点是和信号驱动模型有所不同的，在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用IO函数进行实际的读写操作；而在异步IO模型中，收到信号表示IO操作已经完成，不需要再在用户线程中调用IO函数进行实际的读写操作。

注意，异步IO是需要操作系统的底层支持，在Java7中，提供了Asynchronous IO。

前面四种IO模型实际上都属于同步IO，只有最后一种是真正的异步iO，因为无论是多路复用IO还是信号驱动模型，IO操作的第2个阶段都会引起用户线程的阻塞，也就是内核进行数据拷贝的过程都会让用户线程阻塞。