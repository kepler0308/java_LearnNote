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

