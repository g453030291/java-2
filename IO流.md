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

