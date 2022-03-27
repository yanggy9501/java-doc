# 字符串编码转化

## 1，读取文件的编码转化

> 如文件gbk的读取时要转化为utf-8的编码

```java
/**
* 字符编码的转化：gbk -> utf-8(注意大小写，可能会不对哦)
* @param args
* @throws IOException
*/
public static void main(String[] args) throws IOException {
    // name.json 是gbk编码
    FileInputStream fileio =new FileInputStream("D:\\project\\kafka\\kakfa-start\\src\\main\\resources\\file\\name.json");
    InputStreamReader tInputStreamReader = new InputStreamReader(fileio,"GBK");
    BufferedReader breader = new BufferedReader(tInputStreamReader);
    StringBuffer buf = new StringBuffer();
    while(breader.ready()){
        buf.append((char)breader.read());
    }
    fileio.close();
    tInputStreamReader.close();
    breader.close();
    // 转化为utf-8字符串
    byte[] bytes = buf.toString().getBytes("UTF-8");
    String gbk = new String(bytes);
    //        String gbk = new String(bytes, "UTF-8"); // 可以不指定编码
    System.out.println(gbk);
}
```



## 2，已unicode为中介，实现字符串编码转化

utf-8 ——》unicode——》gbk
gbk ——》unicode——》utf-8
