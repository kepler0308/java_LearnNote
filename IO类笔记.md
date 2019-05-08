# IO类笔记

## 1、文件读写操作

### 文件定义

```java
File file = new File("文件路径");
```

### 字节流（Byte）

```java
InputStream   in = new FileInputStream(file);
outputStream out = new FileOutputStream(file);
```

### 字符流（char）

```java
Reader in = new FileReader(file);
Writer out= new FileWriter(file);
```

操作文本类的文件尽量使用字符流，非文本类文件使用字节流

### 文件复制练习

需要注意读取操作（len = in.read（bytes）！= -1）

### 字符流和字节流的相互转换

![1557149627586](C:\Users\kepler\AppData\Roaming\Typora\typora-user-images\1557149627586.png)

使用InputStreamReader和OutputStreamWriter进行转换

```java
OutputStream out = new FileOutputStream("C:\\java_test\\filetest\\kepler.txt");
Writer writer = new OutputStreamWriter(out,Charset.defaultCharset());//默认编码

InputStream in = new FileInputStream("C:\\java_test\\filetest\\kepler.txt");
Reader read = new InputStreamReader(in,Charset.defaultCharset());//默认编码
```



### 字节缓存流

缓存的目的：解决在写入文件操作时，频繁的操作文件造成性能的下降。

BufferedOutputStream 内部默认缓存大小为8KB，每次写入时存储到缓存的byte数组中，当数组存满后，会把数组中的数据写入文件中，并且缓存下标归零

```java
OutputStream out = new FileOutputStream(file);	//输出缓存流
BufferedOutputStream bos = new BufferedOutputStream(out);
out.close(); //关闭缓存流，不用关闭字符流
	
InputStream in = new FileInputStream(file); 	//输入缓存流
BufferedInputStream bis = new BufferedInputStream(in);
bis.close(); //关闭缓存流，不用关闭字符流

try(BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file)){
    //这样写可以自动关闭流，1.7版本后的新特性
}
```

### 字符缓存流

1、加入字符缓存流，增强读取功能（readLine）

2、更高效的读取数据。

FileReader:内部使用InputStreamReader，解码过程byte-->char，默认缓存大小8K

BufferedReader / BufferedWriter：默认缓存大小为8K，可自定义缓存大小，把数据直接读取到缓存中，减少转换次数，提高效率。

```java
File file = new File("C:\\java_test\\filetest\\kepler.txt");

Writer writer = new FileWriter(file);
BufferedWriter bw = new BufferedWriter(writer);

Reader reader = new FileReader(file);
BufferedReader br = new BufferedReader(reader);
```



### 打印流

```java
File file = new File("C:\\java_test\\filetest\\kepler.txt");

Writer writer = new FileWriter(file);	//字节打印流
BufferedWriter bw = new BufferedWriter(writer);
PrintWriter pw = new PrintWriter(bw);

OutputStream out = new FileOutputStream(file);	//字符打印流
BufferedOutputStream bos = new BufferedOutputStream(out);	//定义缓存流
PrintStream ps = new PrintStream(bos);	//定义打印流
```



### 对象流与序列化

什么时候需要序列化：

1、把对象保存到文件中（存储到物理介质）

2、对象需要在网络上传输时。

对象序列化后，把对象写入文件，实际是写入类名，属性名，属性类型，属性的值等。反序列就是将这些内容读取出来，还原成对象。

```java
Dog dog = new Dog("wangcai",2,"公");		//对象序列化，并写入文件
File file = new File("C:\\dog.obj");
OutputStream out = new FileOutputStream(file);
ObjectOutputStream oos = new ObjectOutputStream(out);
oos.writeObject(dog);
oos.close();

InputStream in = new FileInputStream(file);
ObjectInputStream ois = new ObjectInputStream(in);
Dog dog = ois.readObject(dog);
ois.close();
```

如果一个类创建的对象需要序列化，那么这个类需要实现Serializable接口。如果不实现，程序会抛出NotSerializableException异常。

```java
public class Dog implements Serializable{
    private ...
}
```

使用transient关键字修饰变量，可以使对象存储时，被修饰的变量的值不需要维持（被修饰的值不会被存储）

```java
private transient int id;	//存储时，id的值不会被存储，取出时为int类型默认值0
```



### 字节数组流

字符数组流是基于内存（RAM）操作，内部维护着一个字节数组，可以用来操作数组，非常方便。由于是基于内存的，所以无需关闭，也不会引起IO异常。

```java
String s = "2131455as11qwsd123fdsfJOIION123!@#$%^&IZC";	//通过字节数组流提取字符串中的字母
ByteArrayInputStream bais = new ByteArrayInputStream(s.getBytes());
ByteArrayOutputStream baos = new ByteArrayOutputStream();

int curr = -1;	//定义读取到的数据
//需要注意read方法和之前不同，返回为读取到的数据，而不是读取数据的长度
while((curr = bais.read())!=-1) {	
    if((curr>=65 && curr<=90) || (curr>=97 && curr<=122)) {
        baos.write(curr);
    }
}

//不需要关闭，因为是基于内存（Ram）的，也不会引起IO异常
System.out.println(baos.toString());
System.out.println(baos.toString().length());
```

ByteArrayInputStream类的read()方法如下

```java
public synchronized int read() {	//pos为当前读取长度，count为总长度
	return (pos < count) ? (buf[pos++] & 0xff) : -1;
}
```



### 数据流

数据流可以用来操作与机器无关的java基本数据类型（int char byte boolean ...）

```java
File file = new File("C:\\java_test\\filetest\\data.dat");

OutputStream out = new FileOutputStream(file);		//数据输入流的用法
DataOutputStream dos = new DataOutputStream(out);

dos.writeInt(10);	//4字节
dos.writeInt(14);	//4字节
dos.writeChar(62);	//2字节
dos.writeUTF("云");	//5字节

dos.close();

InputStream in = new FileInputStream(file);
DataInputStream dis = new DataInputStream(in);

int num = dis.readInt();	//注意要按照存的顺序进行取，不然会出现乱码
int num2= dis.readInt();	//如果少取了数据（比如num2不取），会出现EOFException异常
char c  = dis.readChar();
String s= dis.readUTF();

dis.close();
```

