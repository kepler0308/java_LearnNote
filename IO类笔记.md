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

![](https://raw.githubusercontent.com/kepler0308/java_LearnNote/master/img/%E5%AD%97%E7%AC%A6%E6%B5%81%E5%AD%97%E8%8A%82%E6%B5%81%E8%BD%AC%E6%8D%A2.jpg)



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
String s = "2131455as11qwsd123fdN123!@#$%^&IZC";	//通过字节数组流提取字符串中的字母
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



### 文件分割

文件分割主要流程：

1、输入文件路径，需要分割的size。

2、根据文件大小和需要分割的size判断总共需要分割成多少小文件。

3、通过循环生成对应的小文件，并根据分割的size确定读取的字节数组大小及读取的次数

```java
//通过下面的判断，当尺寸较小的时候，不用生成大数组，节省空间
if(cutSize < (1024 * 8)) {
    bytes = new byte[(int) cutSize];
    count = 1;
}else {
    bytes = new byte[1024*8];
    count = (int) (cutSize % (1024 * 8) == 0 ? 
                   cutSize / (1024 * 8) : cutSize / (1024 * 8) + 1);
}
```

4、读取文件数据到字节数组中，然后写入到创建的小文件中。

5、完成操作，之后关闭流。



### 文件合并

使用SequenceInputStream类可以方便的进行流合并

```java
//SequenceInputStream类可以自动的将流合并，es是一个文件输入流的集合
//也可以通过SequenceInputStream(InputStream s1,InputStream s2)构造方法
SequenceInputStream sis = new SequenceInputStream(es);
```



### 字符串流

以一个字符串为数据源，来构造一个字符流

```java
String info = "good good study day day up";
StringReader sr = new StringReader(info);	//构建字符串流
StreamTokenizer st = new StreamTokenizer(sr); //构建流标记器
```

在web开发中，经常需要对字符串进行处理，此时，需要构建字符串流，然后使用第三方的数据解析器来解析数据。



### 管道流

多线程之间的通讯，暂时不学习，因为会用到多线程的知识，学习完多线程后重新学习。



### RandonAccessFile类

随机存取类（也叫直接存取），可以同时进行读写。现在被NIO类替代了，但以前的项目可能会使用，需要了解。具体的使用方法和之前流类似，有一些方法可以对文件进行截取操作，具体需要查阅API文档。

```java
//构造方法第一个参数可以传文件（不会覆盖，自动添加）或者字符串路径（会自动创建文件）
//第二个参数是读写控制位，但只能传参数"r", "rw", "rws", or "rwd"
RandomAccessFile r = new RandomAccessFile("C:\\kepler.txt", "r");
RandomAccessFile w = new RandomAccessFile("C:\\java_test\\kepler.txt","rw");
```



### Properties类（属性文件）

Properties类可以对.properties结尾的属性文件进行读写操作，这样可以通过修改配置文件，来更改程序中的内容，不用重新打包程序。

.properties文件格式如下，一般是“key-value”格式。

```properties
app.version=1.1
db.username=kepler
db.password=12345
```

读操作：

```java
Properties p = new Properties();
InputStream in = new FileInputStream("config.properties");
p.load(in);	//加载属性文件
			
version = p.getProperty("app.version");		//通过key值，读取value
username = p.getProperty("db.username");
password = p.getProperty("db.password");
in.close();	//记得关闭文件流
```

写操作稍微复杂一些：

```java
Properties p = new Properties();
p.put("app.version",version);	//输入需要向属性文件添加的key-value值
p.put("db.username",username);
p.put("db.password",password);

OutputStream out = new FileOutputStream("config.properties");
p.store(out, "config updata"); //写文件
out.close();
```

Tips1:可以将属性文件的读操作放在static代码块，这样代码只会执行一次。

```java
static{
    readConfig();	//方法类型应该是static
}
```

Tips2:当属性文件不再文件的根目录下，而是单独的package下，文件流需要用以下方法定义。

```java
//项目的class文件放在com/kepler下，属性文件放在com/res下，需要用一下方法创建文件流
InputStream in = Thread.currentThread().
	getContextClassLoader().getResourceAsStream("com/res/config.properties");
```



### 文件压缩与解压

通过ZipOutputStream类对文件进行压缩。

```java
//创建压缩文件流，并创建buffer类，提高压缩效率
ZipOutputStream out = new ZipOutputStream(new FileOutputStream(zipFileName));
BufferedOutputStream bos = new BufferedOutputStream(out);
//压缩文件的方法。
//参数为：压缩文件流、需要压缩的文件（或文件夹）、需要压缩的文件（文件夹）名字、压缩文件缓冲流
zip(out,targetFile,targetFile.getName(),bos);	

bos.close();
out.close();
```

zip()方法的流程如下，需要借助递归的方法，流程如下（没装visvo，简单画一下）:

![](https://raw.githubusercontent.com/kepler0308/java_LearnNote/master/img/file_compression.jpg)

通过putNextEntry()方法可以将文件及其目录添加进压缩流，具体代码如下：

```java
private static void zip(ZipOutputStream zOut, File targetFile, String name, BufferedOutputStream bos) throws IOException {
		if(targetFile.isDirectory()) {	//判断为文件夹
			File[] files = targetFile.listFiles();	//读取文件夹中的文件
			
			if(files.length == 0) {		//判断为空文件夹
				zOut.putNextEntry(new ZipEntry(name+"/"));
			}
			for(File f:files) {
				//System.out.println(f.getName());
				zip(zOut,f,name+"/"+f.getName(),bos);		//递归处理
			}
		}else {		//如果是文件
			zOut.putNextEntry(new ZipEntry(name));		//添加到压缩流中
			InputStream in = new FileInputStream(targetFile);
			BufferedInputStream bis = new BufferedInputStream(in);
			
			byte[] bytes = new byte[1024];
			int len = -1;
			while((len = bis.read(bytes)) != -1) {
				bos.write(bytes, 0, len);
			}
			bis.close();
		}
	}
```

文件的解压则通过ZipInputStream类完成，通过getNextEntry方法可以将压缩时输入文件的名称及路径获取出来，具体方法如下:

```java
//targetFileName是需要解压的文件，parent是解压到的文件夹
ZipInputStream zIn = new ZipInputStream(new FileInputStream(targetFileName));
ZipEntry entry = null;	//如果放进循环中会不断创建新的类，所以程序中的类尽量写在文件外面。
File file = null;

//依次读取压缩进去的文件机器目录
while((entry = zIn.getNextEntry())!=null /*&& !entry.isDirectory()*/) {
    file = new File(parent,entry.getName());	//文件的构造方法，参数为父路径和文件名
    if(!file.exists()) {	//判断文件或文件目录不存在
        new File(file.getParent()).mkdirs();	//创建文件和其文件目录
    }
    OutputStream out = new FileOutputStream(file);
    BufferedOutputStream bos = new BufferedOutputStream(out);
    byte[] bytes = new byte[1024];
    int len = -1;
    while((len = zIn.read(bytes)) != -1) {
        bos.write(bytes,0,len);
    }
    bos.close();
    System.out.println(file.getAbsolutePath()+"解压成功");
    }
	zIn.close();
}
```

