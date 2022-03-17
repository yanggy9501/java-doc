# BigDecimal

## 1. 简介

> BigDecimal，用来对超过16位有效位的数进行精确的运算。
>
> float和double只能用来做科学计算或者是工程计算，在商业计算中要用java.math.BigDecimal。

**构造方法：**

* BigDecimal(int)       创建一个具有参数所指定整数值的对象。 
* BigDecimal(double) 创建一个具有参数所指定双精度值的对象。 //不推荐使用，因为double对象就产生了进度问题，当double必须用作BigDecimal的源时，请使用`Double.toString(double)转成String`
* BigDecimal(long)    创建一个具有参数所指定长整数值的对象。 
* **BigDecimal(String)** 创建一个具有参数所指定以字符串表示的数值的对象。**//推荐使用**

| 返回值     | 方法                         | 描述                                            |
| ---------- | ---------------------------- | ----------------------------------------------- |
| BigDecimal | add(BigDecimal b2)           | BigDecimal对象中的值相加，返回新对象。b2被加数  |
| BigDecimal | subtract(BigDecimal b2)      | BigDecimal对象中的值相减，返回新对象。b2被减数  |
| BigDecimal | multiply(BigDecimal b2)      | BigDecimal对象中的值相乘,，返回新对象。b2被乘数 |
| BigDecimal | divide(BigDecimal b2[,....]) | BigDecimal对象中的值相除，返回新对象。b2被除数  |
| String     | toString()                   | 将BigDecimal对象的数值转换成字符串。            |
| double     | doubleValue()                | 将BigDecimal对象中的值以双精度数返回。          |
| float      | floatValue()                 | 将BigDecimal对象中的值以单精度数返回。          |
| long       | longValue()                  | 将BigDecimal对象中的值以长整数返回。            |
| int        | intValue()                   | 将BigDecimal对象中的值以整数返回。              |

**注意divide除法：**

> 如果进行除法运算的时候，结果不能整除，有余数，这个时候会报java.lang.ArithmeticException。
>
> 这边我们要避免这个错误产生，在进行除法运算的时候，针对可能出现的小数产生的计算，必须要多传两个参数。
>
>  **divide(BigDecimal，保留小数点后几位小数，舍入模式)**
>
> **divide(BigDecimal，2，BigDecimal.ROUND_HALF_UP)**
>
> 补充：
>
> 需要对BigDecimal进行截断和四舍五入可用setScale方法。例：
>
> BigDecimal a = new BigDecimal("3.1415927");
>
> a = a.setScale(2, BigDecimal.ROUND_HALF_UP);

**舍入模式：**

> * ROUND_CEILING :向正无穷方向舍入
> * ROUND_DOWN   ://向零方向舍入
> * ROUND_FLOOR    :向负无穷方向舍入
> * ROUND_HALF_UP :四舍五入
> * .......

****

## 2. BigDecimal大小比较

> java中对BigDecimal比较大小一般用的是bigdemical的compareTo方法。
>
> （相对第一个 - 第二个是正是负）

```java
int a = bigdemical.compareTo(bigdemical2)
```

a = -1,表示bigdemical小于bigdemical2；
a = 0,表示bigdemical等于bigdemical2；
a = 1,表示bigdemical大于bigdemical2；



## **3. BigDecimal格式化**

> BigDecimal使用DecimalFormat进行格式化。
>
> 通过DecimalFormat的format方法格式化BigDecimal对象，返回一个格式化后的字符串。
>
> DecimalFormat还提供了其他已经设置了格式的静态方法可以直接使用

```java
 //保留两位小数: 0.00表示没数字位置用0补齐。注意不要填其他数字只能是0
DecimalFormat df1 = new DecimalFormat("0.00"); 
// 最多保留两位小数: #.##代表有就有,没有就没有
DecimalFormat df2 = new DecimalFormat("#.##");
```

**代码：**

```java
BigDecimal b1 = new BigDecimal("1.1");
DecimalFormat df1 = new DecimalFormat("0.00");
String f1 = df1.format(b1);
System.out.println(f1); // 1.10

BigDecimal b2 = new BigDecimal("2.0");
DecimalFormat df2 = new DecimalFormat("#.##");
String f2 = df2.format(b2);
System.out.println(f2); // 2
```

## 4. utils

```java
import java.math.BigDecimal;

/**
 * 用于高精确处理常用的数学运算
 */
public class MathUtils {
    //默认除法运算精度
    private static final int DEF_DIV_SCALE = 10;

    /**
     * 提供精确的加法运算
     *
     * @param v1 被加数
     * @param v2 加数
     * @return 两个参数的和
     */

    public static double add(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2).doubleValue();
    }

    /**
     * 提供精确的加法运算
     *
     * @param v1 被加数
     * @param v2 加数
     * @return 两个参数的和
     */
    public static BigDecimal add(String v1, String v2) {
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.add(b2);
    }

    /**
     * 提供精确的加法运算
     *
     * @param v1    被加数
     * @param v2    加数
     * @param scale 保留scale 位小数
     * @return 两个参数的和
     */
    public static String add(String v1, String v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.add(b2).setScale(scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 提供精确的减法运算
     *
     * @param v1 被减数
     * @param v2 减数
     * @return 两个参数的差
     */
    public static double sub(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2).doubleValue();
    }

    /**
     * 提供精确的减法运算。
     *
     * @param v1 被减数
     * @param v2 减数
     * @return 两个参数的差
     */
    public static BigDecimal sub(String v1, String v2) {
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.subtract(b2);
    }

    /**
     * 提供精确的减法运算
     *
     * @param v1    被减数
     * @param v2    减数
     * @param scale 保留scale 位小数
     * @return 两个参数的差
     */
    public static String sub(String v1, String v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.subtract(b2).setScale(scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 提供精确的乘法运算
     *
     * @param v1 被乘数
     * @param v2 乘数
     * @return 两个参数的积
     */
    public static double mul(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2).doubleValue();
    }

    /**
     * 提供精确的乘法运算
     *
     * @param v1 被乘数
     * @param v2 乘数
     * @return 两个参数的积
     */
    public static BigDecimal mul(String v1, String v2) {
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.multiply(b2);
    }

    /**
     * 提供精确的乘法运算
     *
     * @param v1    被乘数
     * @param v2    乘数
     * @param scale 保留scale 位小数
     * @return 两个参数的积
     */
    public static double mul(double v1, double v2, int scale) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return round(b1.multiply(b2).doubleValue(), scale);
    }

    /**
     * 提供精确的乘法运算
     *
     * @param v1    被乘数
     * @param v2    乘数
     * @param scale 保留scale 位小数
     * @return 两个参数的积
     */
    public static String mul(String v1, String v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.multiply(b2).setScale(scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 提供（相对）精确的除法运算，当发生除不尽的情况时，精确到
     * 小数点以后10位，以后的数字四舍五入
     *
     * @param v1 被除数
     * @param v2 除数
     * @return 两个参数的商
     */

    public static double div(double v1, double v2) {
        return div(v1, v2, DEF_DIV_SCALE);
    }

    /**
     * 提供（相对）精确的除法运算。当发生除不尽的情况时，由scale参数指
     * 定精度，以后的数字四舍五入
     *
     * @param v1    被除数
     * @param v2    除数
     * @param scale 表示表示需要精确到小数点以后几位。
     * @return 两个参数的商
     */
    public static double div(double v1, double v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException("The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.divide(b2, scale, BigDecimal.ROUND_HALF_UP).doubleValue();
    }

    /**
     * 提供（相对）精确的除法运算。当发生除不尽的情况时，由scale参数指
     * 定精度，以后的数字四舍五入
     *
     * @param v1    被除数
     * @param v2    除数
     * @param scale 表示需要精确到小数点以后几位
     * @return 两个参数的商
     */
    public static String div(String v1, String v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException("The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v1);
        return b1.divide(b2, scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 提供精确的小数位四舍五入处理
     *
     * @param v     需要四舍五入的数字
     * @param scale 小数点后保留几位
     * @return 四舍五入后的结果
     */
    public static double round(double v, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException("The scale must be a positive integer or zero");
        }
        BigDecimal b = new BigDecimal(Double.toString(v));
        return b.setScale(scale, BigDecimal.ROUND_HALF_UP).doubleValue();
    }

    /**
     * 提供精确的小数位四舍五入处理
     *
     * @param v     需要四舍五入的数字
     * @param scale 小数点后保留几位
     * @return 四舍五入后的结果
     */
    public static String round(String v, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b = new BigDecimal(v);
        return b.setScale(scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 取余数
     *
     * @param v1    被除数
     * @param v2    除数
     * @param scale 小数点后保留几位
     * @return 余数
     */
    public static String remainder(String v1, String v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        return b1.remainder(b2).setScale(scale, BigDecimal.ROUND_HALF_UP).toString();
    }

    /**
     * 取余数  BigDecimal
     *
     * @param v1    被除数
     * @param v2    除数
     * @param scale 小数点后保留几位
     * @return 余数
     */
    public static BigDecimal remainder(BigDecimal v1, BigDecimal v2, int scale) {
        if (scale < 0) {
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        return v1.remainder(v2).setScale(scale, BigDecimal.ROUND_HALF_UP);
    }

    /**
     * 比较大小
     *
     * @param v1 被比较数
     * @param v2 比较数
     * @return 如果v1 大于v2 则 返回true 否则false
     */
    public static boolean compare(String v1, String v2) {
        BigDecimal b1 = new BigDecimal(v1);
        BigDecimal b2 = new BigDecimal(v2);
        int bj = b1.compareTo(b2);
        boolean res;
        if (bj > 0) {
            res = true;
        } else {
            res = false;
        }
        return res;
    }
}
```

