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