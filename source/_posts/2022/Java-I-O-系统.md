---
title: Java - I/O 系统
date: 2022-09-10 00:46:42
auth: ratears
update: 2022-08-31 02:12:50
categories:
	- [Operating-Systems,I/O]
tags:
	- java
	- io
---



# 认识I/O流

- I/O是Input/Output的缩写， I/O技术是非常实用的技术，用于处理设备之间的数据传输。如读/写文件，网络通讯等

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4rv75wq98ak0.webp" width="60%" />

<br>

<br>

<br>

# Java如何解决I/O问题

- Java将任意数据源或者数据接收端表达为一个具有生成或者接受数据片段能力的对象（以表示“流”的抽象）。Java程序中，对于数据的输入/输出操作以**“流(stream)”** 的方式进行
- 流（Stream）是一个抽象的概念，当程序需要读取数据的时候，就会开启一个通向数据源的流，数据源可以是文件，内存，或是网络连接。反过来，当程序需要写入数据的时候，就会开启一个通向目的地的流。

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5l9vtmu8kk00.webp" width="60%" />

<br>

<br>

- java.io包下提供了各种“流”类和接口，且使用装饰器模式来解决扩展功能的问题，用以获取不同种类的数据，并通过标准的方法输入或输出数据
- Java的IO流共涉及40多个类，实际上非常规则，都是从如下4个抽象基类派生的（由这四个类派生出来的子类名称都是以其父类名作为子类名后缀）

<br>

| （抽象基类） |    字节流    | 字符流 |
| :----------: | :----------: | :----: |
|    输入流    | InputStream  | Reader |
|    输出流    | OutputStream | Writer |

<br>

> Java中所有流流均是由它们派生出来的

<br>

<br>

<br>

# Java I/O 流的历史

- Java 1.0 I/O库诞生，分为输入和输出两类，面向字节。输入相关的类都继承自InputStream，输出相关的类都继承自OutputStream，且整体使用了装饰器的设计模式
- Java 1.1 对I/O库进行了重大的修改，不但增强了面向字节的类库功能，还新增了面向字符的Reader和Writer，以解决国际化的问题，延续了装饰器的设计模式
- Java 1.4 引入了java.nio(new I/O），使用通道（channel），缓冲区（buffer），选择器（Selector）等措施极大的提升了性能
- Java 1.7/1.8 对难用的文件I/O的操作体验进行了巨大的改进，且新增了Asynchronous IO（AIO），这时nio也有了一个别名，称之为non-blocking I/O



> <font color="red">Java8 函数式流和 I/O 流之间并无任何关联</font>

<br>

<br>

<br>

# 流的分类

## 数据单元：字节流和字符流

- 按操作<font color="red">数据单位</font>不同分为：字节流(8 bit)，字符流(16 bit)

<br>

|      类型      | 说明                                                         |
| :------------: | :----------------------------------------------------------- |
| 字节流(8 bit)  | 以字节为单位获取数据，命名上以`Stream`结尾的流一般是字节流，如FileInputStream、FileOutputStream。 字节流可以处理任何一切形式的数据源，包括音频，视频，图片，纯文本，Word，Excel等等 |
| 字符流(16 bit) | 以字符为单位获取数据，命名上以`Reader/Writer`结尾的流一般是字符流，如FileReader、FileWriter。` `字符流只能处理字符串，纯文本等。Java使用Unicode的统一标准字符集，一个字符占用两个字节 |

<br>

<br>

### 字节流（InputStream 和 OutputStream）

- 一切的数据在计算机中都可表示为字节
- 在不同源之间的字节数据的输入与输出，可形象的表示为“字节的流动”，即字节流
- 表示从不同源输入的类：InputStream
  - 文件, 字节数组，字符串对象，管道，其他流，网络等...
- 表示输出到不同源的类：OutputStream

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.29yhndh0ugys.webp" width="80%" />

<br>

<br>

### 字符流（Reader 和 Writer）

- Java 1.0时代的I/O流类库只能支持8位字节流，无法妥善处理16位Unicode字符
- Reader和Writer类并不是为了取代InputStream和OutputStream，而是提供了字符操作的能力，为了在所有的I/O操作中都支持Unicode
- 在需要操作字符的场景中尽量都使用Reader和Writer相关的类，而在需要进行字节操作的场景中，面向字节的InputStream和OutputStream才是正确的选择，比如读取和写入图片文件，java.util.zip库

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6v9r42axexc0.webp" width="80%" />

<br>

<br>

### Unicode

- Unicode(统一码)，它整理和编码了世界上大部分的文字系统，使得电脑可以用更简单的方式呈现和处理文字。它遵循通用字符集 (UCS)并规定了其实现方式，即如何映射为计算机的字节，如何传输等。

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6edn0pzh88g0.webp" width="60%" />

<br>

<br>

### Unicode 与 Java

- Java使用Unicode的统一标准字符集
- Java使用UTF-16编码，所以会将字符串表示为一系列16位的单元，如果标准字符集中字符的数值大于16位（超出U+FFFF），则会拆分为两个16位的单元用以表示一个字符。对于能用16位内数字表示的字符，Unicode的字符数值（Code Point）和UTF-16编码后的16位的单元（Code Unit），是一致的。

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3ab2rmmctv80.webp" width="60%" />

<br>

<br>

## 流的方向：输入流和输出流

- 按数据流的<font color="red">流向</font>不同分为：输入流，输出流

<br>

|  类型  |                           说明                           |
| :----: | :------------------------------------------------------: |
| 输入流 | 数据流向是数据源到程序（以InputStream、Reader结尾的流）  |
| 输出流 | 数据流向是程序到目的地（以OutputStream、Writer结尾的流） |

<br>

<br>

## 处理对象：节点流和处理流

- 按流的<font color="red">角色</font>（处理对象）的不同分为：节点流，处理流

<br>

|  类型  | 说明                                                         |
| :----: | :----------------------------------------------------------- |
| 节点流 | 可以从或向一个特定的地方（节点）读写数据，如FileInputStream、FileReader、DataInputStream等。 （ 没有节点流，处理流发挥不了任何作用。） |
| 处理流 | 不直接连接到数据源或目的地，是 “处理流的流”。通过对已有的节点流进行包装，通过所封装的流的功能调用实现数据读写，提高性能或提高程序的灵活性。 如BufferedInputStream、BufferedReader等。处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。处理流也叫 ”包装流/过滤流“。 |

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7fiok2t2e880.webp" width="60%" />

<br>

<br>

## I/O类型

- 按<font color="red">I/O类型</font>来分类

<br>

|          类型          | 说明                                                         |
| :--------------------: | :----------------------------------------------------------- |
|         文件流         | 对文件进行读、写操作 ：FileReader、FileWriter、FileInputStream、FileOutputStream |
|         缓冲流         | 在读入或写出时，数据可以基于缓冲批量读写，以减少I/O的次数：BufferedReader、BufferedWriter、BufferedInputStream、BufferedOutputStream |
|         内存流         | 1.从/向内存数组读写数据: CharArrayReader、 CharArrayWriter、ByteArrayInputStream、ByteArrayOutputStream 2.从/向内存字符串读写数据 StringReader、StringWriter、StringBufferInputStream |
|         转换流         | 按照一定的编码/解码标准将字节流转换为字符流，或进行反向转换（Stream到Reader,Writer的转换类）：InputStreamReader、OutputStreamWriter |
|         对象流         | 字节流与对象实例相互转换,实现对象的序列化 ：ObjectInputStream、ObjectOutputStream 注意: 1.读取顺序和写入顺序一定要一致，不然会读取出错。 2.在对象属性前面加`transient`关键字，则该对象的属性不会被序列化 |
|         打印流         | 只有输出,没有输入，在整个IO包中，打印流是输出信息最方便的类,分为 PrintWriter（字符打印流）、PrintStream(字节打印流) |
|  DataConversion数据流  | 按基本数据类型读/写，可以字节流与基本类型数据相互转换：DataInputStream、DataOutputStream |
|         过滤流         | 在数据进行读或写时进行过滤：FilterReader、FilterWriter、FilterInputStream、FilterOutputStream |
|         合并流         | 把多个输入流按顺序连接成一个输入流 ：SequenceInputStream     |
|      操作ZIP包流       | ZipInputStream可以读取zip格式的流，ZipOutputStream可以把多份数据写入zip包 |
|      操作JAR包流       | JarInputStream/JarOutputStream,派生自ZipInputStream/ZipOutputStream，它增加的主要功能是直接读取jar文件里面的MANIFEST.MF文件。因为本质上jar包就是zip包，只是额外附加了一些固定的描述文件。 |
|         管道流         | 线程交互的时候使用，管道输出流可以连接到管道输入流，以创建通信管道。管道输出流是管道的发送端。通常数据由某个线程写入管道输出流，并由其他线程从连接的管道输入流读取。注意，管道输出流和输入流需要对接。: PipedReader、PipedWriter、PipedInputStream、PipedOutputStream |
|      Counting计数      | 在读入数据时对行记数 ：LineNumberReader、LineNumberInputStream |
|       推回输入流       | 通过缓存机制，进行预读 ：PushbackReader、PushbackInputStream |
| 接收和响应客户端请求流 | servletinputstream：用来读取客户端的请求信息的输入流 servletoutputstream:可以将数据返回到客户端 |
|     随机读取写入流     | RandomAccessFile 既可以读取文件内容，也可以向文件输出数据，RandomAccessFile 对象包含一个记录指针，标识当前读写处的位置，可以控制记录指针从IO任何位置读写文件 |
|         加密流         | 对流加密/解密 CipherOutputStream 由一个 OutputStream 和一个 Cipher 组成 ,write() 方法在将数据写出到基础 OutputStream 之前先对该数据进行处理(加密或解密) , 同样CipherInputStream是由InputStream和一个Cipher组成,read()方法在读入时,对数据进行加解密操作 |
|       数字签名流       | DigestInputStream : 最大的特点是在读取的数据的时候已经调用MessageDigest实例的update方法，当数据从底层的数据流中读取之后就只可以直接调用MessageDigest实例的digest()方法了，从而完成对输入数据的摘要加密 DigestOutputStream :最大的特点是在向底层的输出流写入数据的时候已经调用MessageDigest实例的update方法，并作为MessageDigest的输入数据，之后就可以直接调用MessageDigest实例的digest()方法完成加密过程；同样的，是否对数据加密也是由该流的on(boolean b)方法进行控制的，如果设置成false，那么在写出数据的过程中便不会将数据传给update方法，那么此时它跟普通的输出流就没有任何区别了 |

<br>

> *CipherInputStream和CipherOutputStream与DigestInputStream/DigestOutputStream/类似，只是后者更为彻底，它们不用在显示地调用传入的Cipher对象的update和doFinal方法，加密或解密过程在读写数据的同时已经隐式地完成了*

<br>

<br>

<br>

# I/O 流体系

## I/O流体系图

- 下图基于Java 1.8制作，其中需要注意的是`StringBufferInputStream`和`LineNumberInputStream`已被废弃

> `StringBufferInputStream`: 该类无法准确的将字符转换为字节，推荐用`StringReader`来取代使用
>
> `LineNumberInputStream`: 该类错误地认为字节能恰当地表示字符，推荐使用字符流的类来取代，即`LineNumberReader`

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.fv0bu32iqn4.webp" width="80%" />

<br><br>

## I/O流体系类库

|      分类      |        字节输入流        |        字节输出流         |     字符输入流      |     字符输出流      |
| :------------: | :----------------------: | :-----------------------: | :-----------------: | :-----------------: |
|   *抽象基类*   |      *InputStream*       |      *OutputStream*       |      *Reader*       |      *Writer*       |
|  **访问文件**  |   **FileInputStream**    |   **FileOutputStream**    |   **FileReader**    |   **FileWriter**    |
|  **访问数组**  | **ByteArrayInputStream** | **ByteArrayOutputStream** | **CharArrayReader** | **CharArrayWriter** |
|  **访问管道**  |   **PipedInputStream**   |   **PipedOutputStream**   |   **PipedReader**   |   **PipedWriter**   |
| **访问字符串** |                          |                           |  **StringReader**   |  **StringWriter**   |
|     缓冲流     |   BufferedInputStream    |   BufferedOutputStream    |   BufferedReader    |   BufferedWriter    |
|     转换流     |                          |                           |  InputStreamReader  | OutputStreamWriter  |
|     对象流     |    ObjectInputStream     |    ObjectOutputStream     |                     |                     |
|   *抽象基类*   |   *FilterInputStream*    |   *FilterOutputStream*    |   *FilterReader*    |   *FilterWriter*    |
|     打印流     |                          |        PrintStream        |                     |     PrintWriter     |
|   推回输入流   |   PushbackInputStream    |                           |   PushbackReader    |                     |
|     特殊流     |     DataInputStream      |     DataOutputStream      |                     |                     |

<br>

> 注：表中**粗体**所表示的类代表节点流。*斜体* 表示的类代表抽象基类，无法直接创建实例。其他的为处理流

<br>

<br>

## RandomAccessFile

- 自成体系，Java输入/输出流体系中功能最丰富的文件内容访问类，既可以读取文件内容，也可以向文件输出数据。
- 不支持装饰器，无法与InputStream/OutputStream联合起来用
- 与普通的输入/输出流不同的是，RandomAccessFile支持跳到文件任意位置读写数据（支持读写随机文件），可将文件视为在磁盘上的一个大的字节数组，我们能通过数组下标（文件指针）来访问里面的内容
- RandomAccessFile对象包含一个记录指针，标识当前读写处的位置，当程序创建一个新的RandomAccessFile对象时，该对象的文件记录指针对于文件头（也就是0处），当读写n个字节后，文件记录指针将会向后移动n个字节，RandomAccessFile可以通过seek()方法自由移动记录指针
- 使用RandomAccessFile就必须知道文件的布局，确定要操作的位置
- 优先考虑使用nio的内存映射

<br>

<br>

### RandomAccessFile使用

- 操作文件记录指针

```java
// 返回文件记录指针的当前位置
long getFilePointer()
    
// 将文件记录指针定位到pos位置
void seek(long pos)
```

<br>

- RandomAccessFile类在创建对象时，除了指定文件本身，还需要指定一个`mode`参数。该参数指定RandomAccessFile的访问模式，该参数有如下四个值：

> 1. r：以只读方式打开指定文件。如果试图对该RandomAccessFile指定的文件执行写入方法则会抛出IOException
> 2. rw：以读取、写入方式打开指定文件。如果该文件不存在，则尝试创建文件
> 3. rws：以读取、写入方式打开指定文件。相对于rw模式，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备，默认情形下(rw模式下),是使用buffer的,只有cache满的或者使用RandomAccessFile.close()关闭流的时候儿才真正的写到文件
> 4. rwd：与rws类似，只是仅对文件的内容同步更新到磁盘，而不修改文件的元数据

<br>

<br>

## I/O流的主要类

- 整个Java.io包中最重要的就是5个类和一个接口

<br>

<br>

### 5个类：

|      类      |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|     File     | 表示一个文件或者目录,可以获取文件或目录相关属性,以及创建文件或目录 |
| InputStream  |  字节输入流父类，单位为字节，定义了所有字节输入流的基本操作  |
| OutputStream |  字节输出流父类，单位为字节，定义了所有字节输出流的基本操作  |
|    Reader    |  字符输入流父类，单位为字符，定义了所有字符输入流的基本操作  |
|    Writer    |  字符输出流父类，单位为字符，定义了所有字符输出流的基本操作  |

<br>

> Java中所有流均是由它们派生出来的
>
> **jdk 1.4**版本开始引入了新I/O类库，它位于 `java.nio` 包中，新I/O类库利用通道和缓冲区等来提高I/O操作的效率

<br>

<br>

### 1个接口：

|      类      |                     说明                     |
| :----------: | :------------------------------------------: |
| Serializable | 序列化/反序列化对象需要实现 Serializable接口 |

<br>

<br>

## I/O流主要三个部分

| 部分       |                             说明                             |
| ---------- | :----------------------------------------------------------: |
| 流式部分   |                        I/O的主体部分                         |
| 非流式部分 | 主要包含一些辅助流式部分的类，如：`File`类、`RandomAccessFile`类和`FileDescriptor`等类。 `RandomAccessFile`（随机读取和写入流）可以从文件的任意位置进行读写） |
| 其他类     | 文件读取部分的与安全相关的类，如：`SerializablePermission`类，以及与本地操作系统相关的文件系统的类，如：`FileSystem`类和`Win32FileSystem`类和`WinNTFileSystem`类 |

<br>

<br>

## 重要I/O流的解读

### 转换流





<br>

### 标准输入、输出流





<br>

### 打印流



<br>

### 数据流



<br>

### 对象流



<br>

### 随机存取文件流





<br>

<br>

<br>

# I/O流API实践

## 字符流：FileReader - FileWriter

```java
 /**
     * 从某个文件的内容写入到另一个文件
     *  1. read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
     *     read(char[] cbuf):返回每次读入cbuf数组中的字符的个数。如果达到文件末尾，返回-1
     *  2. 异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
     *  3. 读入的文件一定要存在，否则就会报FileNotFoundException。
     *
     * 1. 输出操作，对应的File可以不存在的。并不会报异常
     * 2.
     *          File对应的硬盘中的文件如果不存在，在输出的过程中，会自动创建此文件。
     *          File对应的硬盘中的文件如果存在：
     *                 如果流使用的构造器是：FileWriter(file,false) / FileWriter(file):对原有文件的覆盖
     *                 如果流使用的构造器是：FileWriter(file,true):不会对原有文件覆盖，而是在原有文件基础上追加内容
     *  3. 流的关闭顺序，没有严格顺序
     */
    @Test
    public void testFileReaderFileWriter(){
        FileReader fr = null;
        FileWriter fw = null;
        try {
            // 1.File类的实例化
            // 2.流实例化
            fr = new FileReader(new File("hello.txt"));
            fw = new FileWriter(new File("hello2.txt"),true);

            char [] cbuf = new char[5];
            int len;
            // 3.流操作-读入 写出
            while ((len = fr.read(cbuf)) != -1){
                fw.write(cbuf,0,len);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }finally {
            //4.关闭流资源
            //方式一：
            try {
                if(fw != null)
                    fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }finally{
                try {
                    if(fr != null)
                        fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            //方式二：
//            if (null != fw){
//                try {
//                    fw.close();
//                } catch (IOException e) {
//                    throw new RuntimeException(e);
//                }
//            }
//            if (null != fr) {
//                try {
//                    fr.close();
//                } catch (IOException e) {
//                    throw new RuntimeException(e);
//                }
//            }
        }

    }
```

<br>

<br>

## 字节流：FileInputStream - FileOutputStream

- 使用 FileInputStream、FileOutputStream实现文件复制

```shell
	//指定路径下文件的复制
	// 使用字节流FileInputStream处理文本文件，可能出现乱码。
    public void copyFile(String srcPath,String destPath){
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            // File 实例化
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);

            // 造流
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(destFile);

            // 流操作 - 复制的过程
            byte[] buffer = new byte[1024];
            int len;
            while((len = fis.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fos != null){
                //
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
```

<br>

<br>

## 缓冲流：BufferedInputStream - BufferedOutputStream

- 使用 BufferedInputStream 、BufferedOutputStream实现文件复制

```java
	public void testBuffered (){
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;
        try {
            //造文件
            File srcFile = new File("hello.txt");
            File distFile = new File("hello_word.txt");

            //造流
            FileInputStream fis = new FileInputStream(srcFile);
            FileOutputStream fos = new FileOutputStream(distFile);

            //造缓冲流
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);

            //3.复制的细节：读取、写入
            byte [] buffered = new byte[10];
            int len;
            while ((len=bis.read(buffered)) != -1){
                bos.write(buffered,0,len);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            //4.资源关闭
            //要求：先关闭外层的流，再关闭内层的流
            //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
            if(bos != null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(bis != null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
```

- 用BufferedReader和BufferedWriter实现文本文件的复制

```java
    public void testBufferedReaderBufferedWriter(){
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            //创建文件和相应的流
            br = new BufferedReader(new FileReader(new File("dbcp.txt")));
            bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));

            //读写操作
            //方式一：使用char[]数组
//            char[] cbuf = new char[1024];
//            int len;
//            while((len = br.read(cbuf)) != -1){
//                bw.write(cbuf,0,len);
//    //            bw.flush();
//            }

            //方式二：使用String
            String data;
            while((data = br.readLine()) != null){
                //方法一：
//                bw.write(data + "\n");//data中不包含换行符
                //方法二：
                bw.write(data);//data中不包含换行符
                bw.newLine();//提供换行的操作

            }


        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            if(bw != null){

                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
```

<br>

<br>

## 转换流

- 综合使用InputStreamReader和OutputStreamWriter

```java
	public void test2() throws Exception {
        //1.造文件、造流
        File file1 = new File("dbcp.txt");
        File file2 = new File("dbcp_gbk.txt");

        FileInputStream fis = new FileInputStream(file1);
        FileOutputStream fos = new FileOutputStream(file2);

        InputStreamReader isr = new InputStreamReader(fis,"utf-8");
        OutputStreamWriter osw = new OutputStreamWriter(fos,"gbk");

        //2.读写过程
        char[] cbuf = new char[20];
        int len;
        while((len = isr.read(cbuf)) != -1){
            osw.write(cbuf,0,len);
        }

        //3.关闭资源
        isr.close();
        osw.close();

    }
```









# 参考与延伸

- [IO流体系基本概念以及常用操作 - 探索字符串](https://string.quest/read/1143888#IO_110)
- 并发编程网：[Java IO教程](http://ifeve.com/java-io/)



# 学习备注

> 1. 本文笔记主要根据 尚硅谷的java体系课程的io部分 梳理的
> 2. 后期 应该对io 体系进行一个更大的梳理，包括java的 file path，还有根据 io ，nio bio aio等进行梳理，要明白自己学习的是哪种流，所以要熟悉io历史

<img src="" width="60%" />

<br>