#java.io
http://www.open-open.com/lib/view/open1394876193478.html#articleHeader3
##字节流输入
父类
InputStream
基本字节流
ByteArrayInputStream  读字节从字节数组
FileInputStream   读字节从文件中
ObjectInputStream     包装读出类对象 
PipedInputStream   进程间管道通信
SequenceInputStream 合并输入流
字节装饰流父类
FilterInputStream 
字节装饰流子类
BufferedInputStream   带有缓冲区  读入字节
DataInputStream   包装读入基本数据类型
##字节流输出
父类
OutputStream
基本字节流
ByteArrayOutputStream  输出字节到字节数组
FileOutputStream  输出字节到文件中
ObjectOutputStream 包装输出类对象
PipedOutputStream 进程间管道通信
字节装饰流父类 
FilterOutputStream
字节装饰流子类
BufferedOutputStream 带有缓冲区  输出字节
DataOutputStream 包装输出基本数据类型
PrintStream 输出到指定的介质中
##字节流总结
InputStream是所有的输入字节流的父类，它是一个抽象类。
ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。
PipedInputStream 是从与其它线程共用的管道中读取数据，与Piped 相关的知识后续单独介绍。
ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）。
-------------
OutputStream 是所有的输出字节流的父类，它是一个抽象类。
ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。
PipedOutputStream 是向与其它线程共用的管道中写入数据，
ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流。
##字符流输入
Reader  父类
CharArrayReader  基本介质类
StringReader  基本介质类
PipedReader   基本介质类
BufferedReader 带有缓冲区 装饰类
FilterReader  装饰类父类 （抽象）
InputStreamReader  字节转字符的转换类（抽象）
FileReader 实现字节转字符
##字符流输出
Writer   父类       
StringWriter 基本介质类
CharArrayWriter 基本介质类
PipedWriter 基本介质类
BufferedWriter 带有缓冲区 装饰类
PrintWriter 
FilterWriter 装饰类父类 （抽象）
OutputStreamWriter 字符转字节的转换类（抽象）
FileWriter 实现字符转字节
##字符流总结
Reader 是所有的输入字符流的父类，它是一个抽象类。
CharReader、StringReader 是两种基本的介质流，它们分别将Char 数组、String中读取数据。
PipedReader 是从与其它线程共用的管道中读取数据。
BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。
FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。
InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader 可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。
##其他
PrintWriter 和 PrintStream
RandomAccessFile
File
字符流和字节流的使用场景