# Java NIO概述
## Java NIO核心部分
- Channels
- Buffers
- Selectors
## Channel和Buffer
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

## Channel实例
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
            // 反转Buffer
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()) {
                System.out.println((char) byteBuffer.get());
            }
            // 清除Buffer
            byteBuffer.clear();
            read = fileChannel.read(byteBuffer);
        }
    }
```