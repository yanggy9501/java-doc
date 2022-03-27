# BufferedReader(缓冲区读取内容，避免中文乱码)

```java
public class BufferedReader extends Reader
// 所以BufferedReader有Reader的特性即read方法
```

**构造方法**

* public BufferedReader(Reader in) 构造方法 接收一个Reader类的实例
* public BufferedReader(Reader in, int size) 构造方法 接收一个Reader类的实例, size 缓冲区大小



**方法**

* .readLine()  返回值 String

  > 每次读取一行，返回该行字符串，当读取完时，返回null

* .read()    返回值字符的编码，空则返回 -1

  > 抽象类Reader定义的方法，一次读取一个字符，返回该字符的数字编码，若没有数据则返回 -1

* read(char[])  返回值：读取字符的个数，完则返回 -1

  > 一次读取最多char数组大小的字符串，返回读取字符的个数，下一次读取是从上一次的位置读取的



# 类加载器读取文件

> 在测试环境下读取文件可以使用绝对路径，但是一旦打包或者换了一个目录，按绝对路径读取就好包文件找不到异常。解决方法通过类加载器读取文件，其他文件被加载到了target/classes编译目录下

```java
URL resource = TestReadJson.class.getClassLoader().getResource("file/name.json");
FileReader fileReader = new FileReader(resource.getPath());
BufferedReader bufferedReader = new BufferedReader(fileReader);
StringBuffer stringBuffer = new StringBuffer();
String s = bufferedReader.readLine();
while(s != null) {
    stringBuffer.append(s);
    s = bufferedReader.readLine();
}
System.out.println(stringBuffer.toString());
```

