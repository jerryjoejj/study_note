# Java NIO概述
## Java NIO核心部分
- Channels
- Buffers
- Selectors
<<<<<<< HEAD
### Channel和Buffer
=======
## Channel和Buffer
>>>>>>> 0ce6e939a0d56ecdde69d554bc736f748e672fa2
![20190122230739-image.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/20190122230739-image.png)

主要Channel实现
- FileChannel：从文件中读取数据
- DatagramChannel：通过UDP读写网络中的数据
- SocketChannel：通过TCP读写网络中的数据
- ServerSockerChannel：监听新进来的TCP连接

主要Buffer实现
- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- MappedByteBuffer: 内存映射文件

## Selector
Selector允许单线程处理多个Channel。

![20190122231236-image.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/20190122231236-image.png)

要使用Selector需要Selector注册Channel

<<<<<<< HEAD
### Channel实例
=======
## Channel实例
>>>>>>> 0ce6e939a0d56ecdde69d554bc736f748e672fa2
```java
    public void testChannel() throws IOException {
        // 读取文件
        RandomAccessFile file = new RandomAccessFile("D:\\1.txt", "rw");
        // 获取一个文件Channel
        FileChannel fileChannel = file.getChannel();
        // 分类一个字节Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(128);
        // 把Channel的数据读取到Buffer
        int read = fileChannel.read(byteBuffer);
        while (read != -1) {
            System.out.println("Read" + read);
<<<<<<< HEAD
            // 反转Buffer，将Buffer变成可读模式
=======
            // 反转Buffer
>>>>>>> 0ce6e939a0d56ecdde69d554bc736f748e672fa2
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()) {
                System.out.println((char) byteBuffer.get());
            }
            // 清除Buffer
            byteBuffer.clear();
            read = fileChannel.read(byteBuffer);
        }
    }
<<<<<<< HEAD
```
# Buffer
Java NIO的Buffer用于和NIO通道进行交互，数据从通道读入缓冲区，从缓冲区写入通道
## Buffer基本用法
Buffer使用步骤：
- 1.写入数据到Buffer
- 2.调用filp()方法，将Buffer从写模式切换到读模式
- 3.从Buffer中读取数据
- 4.调用clear()方法或者compact()方法清空缓冲区。clear()方法会清空整个缓冲区，compact()只会清除已读数据，任何未读数据都会被移动到缓冲区的起始处，新写入的数据将放到缓冲区未读数据后面。
## Buffer的limit、capacity、position
![20190123125808-buffers-modes.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/20190123125808-buffers-modes.png)
### capacity
Buffer内存块的固有大小，只可以向Buffer中写入capacity个byte、long、char等类型数据，一旦Buffer满了就需要清空Buffer才能继续向Buffer中写入数据。
### position
position表示当前位置。初始position为0，写入一个数据（Buffer对应的类型）到Buffer中，position会向前移动下一个 可插入数据的Buffer单元，position最大值为capacity-1。当Buffer从写模式切换到读模式，position会被重置为0。当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
### limit
在写模式下，Buffer的limit表示最多可以向Buffer中写入的数据，limit=capacity。在读模式下，limit表示最多能读到多少数据，limit=写模式下的position。
## Buffer类型
- ByteBuffer
- MapperByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
## Buffer的分配
每一个Buffer都有allocate方法。<br>
分配一个capacity为48个字节的ByteBuffer

```java
ByteBuffer buf = ByteBuffer.allocate(48);
```

分配一个可存储1024个字符的CharBuffer
```java
CharBuffer buf = CharBuffer.allocate(1024);
```
## 向Buffer中写入数据
向Buffer中写入数据有两种方法
- 1.从Channel写入到Buffer
- 2.通过Buffer的put方法写入Buffer

通过Channel写入的例子
```java
int read = fileChannel.read(byteBuffer);
```

通过put()方法写入的例子。put方法有很多类型，可以通过不同的方式把数据写入Buffer
```java
buf.put(127);
```

## filp()方法
调用filp方法会将Buffer从读模式切换到写模式，将position设置为0，将limit设置为调用前position的值。也就是说position现在用于标记读的位置，limit表示之前写入了多少数据。
## 从Buffer中读取数据
有以下两种方式：
- 1.从Buffer读取数据到Channel
```java
int bytesWritten = inChannel.write(buf);
```
- 2.使用get()方法从Buffer中读取数据
```java
byte aByte = buf.get();
=======
>>>>>>> 0ce6e939a0d56ecdde69d554bc736f748e672fa2
```