## Java IO简介

- 在java 1.0中，java提供的类都是以字节(byte)为单位，例如，FileInputStream和FileOutputStream。
- 在java 1.1，为了与国际化进行接轨，在java io中添加了许多以字符(Unicode)为单位进行操作的类。


数据的单位：字节流和字符流。
- 在java中，字节是占1个Byte，即8位；有符号类型。
- 字符是占2个Byte，即16位。无符号类型。

### 以字节为单位的IO流 
##### InputStream / OutputStream
是以字节为单位的输入 输出流的超类。提供了read() / write() 接口从输入流中读取字节数据。
- ByteArrayInputStream 是字节数组输入流。它包含一个内部缓冲区，该缓冲区包含从流中读取的字节；通俗点说，它的内部缓冲区就是一个字节数组，而ByteArrayInputStream本质就是通过字节数组来实现的。
- PipedInputStream / PipedOutputStream 是管道输入输出流，能实现多线程间的管道通信。

##### FilterInputStream / FilterOutputStream 
是过滤输入输出流。

它是DataInputStream / DataOutputStream / BufferedInputStream / BufferedOutputStream 和 PrintStream 的超类。
- DataInputStream / DataOutputStream 是数据输入输出流。它是用来装饰其它输入输出流，它“允许应用程序以与机器无关方式从底层输入流中读取/写入基本 Java 数据类型”。
- BufferedInputStream / BufferedOutputStream 是缓冲输入输出流。它的作用是为另一个输入流添加缓冲功能。
- PrintStream 是打印输出流。它是用来装饰其它输出流，能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。

##### File
是“文件”和“目录路径名”的抽象表示形式。关于File，注意两点：

    - File不仅仅只是表示文件，它也可以表示目录！
    - File虽然在io保重定义，但是它的超类是Object，而不是InputStream。
- FileDescriptor 是“文件描述符”。它可以被用来表示开放文件、开放套接字等。
- FileInputStream / FileOutputStream 是文件输入流。它通常用于对文件进行读取操作。

##### ObjectInputStream / ObjectOutputStream
是对象输入流。用来提供对“基本数据或对象”的持久存储。

### 以字符为单位的IO流 
Reader / Writer 是以字符为单位的输出流的超类。它提供了read() / write()接口往其中读取、写入数据。

- CharArrayReader / CharArrayWriter 是字符数组输入输出流。它用于读取、写入字符数组，它继承于Reader /Writer。操作的数据是以字符为单位！
- PipedReader / PipedWriter 是字符类型的管道输入输出流。可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedWriter配套使用。
- FileReader / FilterWriter 是字符类型的过滤输入输出流。
- BufferedReader / BufferedWriter 是字符缓冲输入输出流。它的作用是为另一个输入输出流添加缓冲功能。
- InputStreamReader / OutputStreamWriter 是字节转字符的输入输出流。它是字节流通向字符流的桥梁：它使用指定的 charset 将字节转换为字符并写入。
- FileReader / FileWriter 是字符类型的文件输入输出流。它通常用于对文件进行读取写入操作。
- PrintWriter 是字符类型的打印输入输出流。它是用来装饰其它输出流，能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。