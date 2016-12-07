---
title: java 中的Double Float BigDecimal 等精度问题
date: 2016-12-06 18:55:12
updated: 2016-12-06 18:55:12
tags:
- java float double BigDecimal
categories:
- java
---

浮点数判断大小有问题，因为底层的二进制数不能精确表示所有的小数。有时候会产生让人觉得莫名其妙的事情。

如在java中，

0.99999999f==1f //true

0.9f==1f //false

要明白这些，首先要搞清楚float和double在内存结构

### 内存结构

float和double的范围是由指数的位数来决定的。

float的指数位有8位，而double的指数位有11位，分布如下：

float：

1bit（符号位） 8bits（指数位） 23bits（尾数位）

double：

1bit（符号位） 11bits（指数位） 52bits（尾数位）

于是，float的指数范围为-128~+127，而double的指数范围为-1024~+1023，并且指数位是按补码的形式来划分的。

其中负指数决定了浮点数所能表达的绝对值最小的非零数；而正指数决定了浮点数所能表达的绝对值最大的数，也即决定了浮点数的取值范围。

float的范围为-2^128 ~ +2^127，也即-3.40E+38 ~ +3.40E+38；double的范围为-2^1024 ~ +2^1023，也即-1.79E+308 ~ +1.79E+308。

> 补充

正数：原码＝反码＝补码；负数二进制取反加1 ,如: 
补码:1101001 
－－－－－－－－－－－－－－－－－－－－ 
原码:0010110 + 1 ＝0010111
### 精度

float和double的精度是由尾数的位数来决定的。浮点数在内存中是按科学计数法来存储的，其整数部分始终是一个隐含着的“1”，由于它是不变的，故不能对精度造成影响。

float：2^23 = 8388608，一共七位，由于最左为1的一位省略了，这意味着最多能表示8位数： 2*8388608 = 16777216 。有8位有效数字，但绝对能保证的为7位，也即 float的精度为7~8位有效数字 ；

double：2^52 = 4503599627370496，一共16位，同理， double的精度为16~17位
。

之所以不能用f1==f2来判断两个数相等，是因为虽然f1和f2在可能是两个不同的数字，但是受到浮点数表示精度的限制，有可能会错误的判断两个数相等！

我们可以用下面这段代码检验一下：
```
float f1 = 16777215f;
for (int i = 0; i < 10; i++) {
    System.out.println(f1);
    f1++;
}
```

### 浮点数运算产生的精度问题

#### BCD码

BCD码（Binary-Coded Decimal）亦称二进码十进数或二-十进制代码。用4位二进制数来表示1位十进制数中的0~9这10个数码。是一种二进制的数字编码形式，用二进制编码的十进制代码。BCD码这种编码形式利用了四个位元来储存一个十进制的数码，使二进制和十进制之间的转换得以快捷的进行。这种编码技巧最常用于会计系统的设计里，因为会计制度经常需要对很长的数字串作准确的计算。相对于一般的浮点式记数法，采用BCD码，既可保存数值的精确度，又可免去使电脑作浮点运算时所耗费的时间。此外，对于其他需要高精确度的计算，BCD编码亦很常用。
由于十进制数共有0、1、2、……、9十个数码，因此，至少需要4位二进制码来表示1位十进制数。4位二进制码共有2^4=16种码组，在这16种代码中，可以任选10种来表示10个十进制数码，共有N=16！/[10!*（16-10）！]等于8008种方案。常用的BCD代码列于末。


![https](pic01.png)

#### 问题的提出
如果我们编译运行下面这个程序会看到什么？
```
public class Test{
    public static void main(String args[]){
        System.out.println(0.05+0.01);
        System.out.println(1.0-0.42);
        System.out.println(4.015*100);
        System.out.println(123.3/100);
    }
};
```
你没有看错！结果确实是
0.060000000000000005
0.5800000000000001
401.49999999999994
1.2329999999999999

Java中的简单浮点数类型float和double不能够进行运算。不光是Java，在其它很多编程语言中也有这样的问题。在大多数情况下，计算的结果是准确的，但是多试几次（可以做一个循环）就可以试出类似上面的错误。现在终于理解为什么要有BCD码了。
这个问题相当严重，如果你有9.999999999999元，你的计算机是不会认为你可以购买10元的商品的。
在有的编程语言中提供了专门的货币类型来处理这种情况，但是Java没有。现在让我们看看如何解决这个问题。
 
四舍五入
我们的第一个反应是做四舍五入。Math类中的round方法不能设置保留几位小数，我们只能象这样（保留两位）：
public double round(double value){
    return Math.round(value*100)/100.0;
}
非常不幸，上面的代码并不能正常工作，给这个方法传入4.015它将返回4.01而不是4.02，如我们在上面看到的
4.015*100=401.49999999999994
因此如果我们要做到精确的四舍五入，不能利用简单类型做任何运算
java.text.DecimalFormat也不能解决这个问题：
System.out.println(new java.text.DecimalFormat("0.00").format(4.025));
输出是4.02
 
BigDecimal
在《Effective Java》这本书中也提到这个原则，float和double只能用来做科学计算或者是工程计算，在商业计算中我们要用 java.math.BigDecimal。BigDecimal一共有4个够造方法，我们不关心用BigInteger来够造的那两个，那么还有两个，它们是：
BigDecimal(double val) 
          Translates a double into a BigDecimal. 
BigDecimal(String val) 
          Translates the String repre sentation of a BigDecimal into a BigDecimal.
上面的API简要描述相当的明确，而且通常情况下，上面的那一个使用起来要方便一些。我们可能想都不想就用上了，会有什么问题呢？等到出了问题的时候，才发现上面哪个够造方法的详细说明中有这么一段：
Note: the results of this constructor can be somewhat unpredictable. One might assume that new BigDecimal(.1) is exactly equal to .1, but it is actually equal to .1000000000000000055511151231257827021181583404541015625. This is so because .1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the long value that is being passed in to the constructor is not exactly equal to .1, appearances nonwithstanding. 
The (String) constructor, on the other hand, is perfectly predictable: new BigDecimal(".1") is exactly equal to .1, as one would expect. Therefore, it is generally recommended that the (String) constructor be used in preference to this one.
原来我们如果需要精确计算，非要用String来够造BigDecimal不可！在《Effective Java》一书中的例子是用String来够造BigDecimal的，但是书上却没有强调这一点，这也许是一个小小的失误吧。
#### 解决方案
现在我们已经可以解决这个问题了，原则是使用BigDecimal并且一定要用String来够造。
但是想像一下吧，如果我们要做一个加法运算，需要先将两个浮点数转为String，然后够造成BigDecimal，在其中一个上调用add方法，传入另一个作为参数，然后把运算的结果（BigDecimal）再转换为浮点数。你能够忍受这么烦琐的过程吗？下面我们提供一个工具类Arith来简化操作。它提供以下静态方法，包括加减乘除和四舍五入：
public static double add(double v1,double v2)
public static double sub(double v1,double v2)
public static double mul(double v1,double v2)
public static double div(double v1,double v2)
public static double div(double v1,double v2,int scale)
public static double round(double v,int scale)
附录
源文件Arith.java：

``` java
import java.math.BigDecimal;
/**
 * 由于Java的简单类型不能够精确的对浮点数进行运算，这个工具类提供精
 * 确的浮点数运算，包括加减乘除和四舍五入。
 */
public class Arith{
    //默认除法运算精度
    private static final int DEF_DIV_SCALE = 10;
    //这个类不能实例化
    private Arith(){
    }
 
    /**
     * 提供精确的加法运算。
     * @param v1 被加数
     * @param v2 加数
     * @return 两个参数的和
     */
    public static double add(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2).doubleValue();
    }
    /**
     * 提供精确的减法运算。
     * @param v1 被减数
     * @param v2 减数
     * @return 两个参数的差
     */
    public static double sub(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2).doubleValue();
    } 
    /**
     * 提供精确的乘法运算。
     * @param v1 被乘数
     * @param v2 乘数
     * @return 两个参数的积
     */
    public static double mul(double v1,double v2){
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.multiply(b2).doubleValue();
    }
 
    /**
     * 提供（相对）精确的除法运算，当发生除不尽的情况时，精确到
     * 小数点以后10位，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @return 两个参数的商
     */
    public static double div(double v1,double v2){
        return div(v1,v2,DEF_DIV_SCALE);
    }
 
    /**
     * 提供（相对）精确的除法运算。当发生除不尽的情况时，由scale参数指
     * 定精度，以后的数字四舍五入。
     * @param v1 被除数
     * @param v2 除数
     * @param scale 表示表示需要精确到小数点以后几位。
     * @return 两个参数的商
     */
    public static double div(double v1,double v2,int scale){
        if(scale<0){
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.divide(b2,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
 
    /**
     * 提供精确的小数位四舍五入处理。
     * @param v 需要四舍五入的数字
     * @param scale 小数点后保留几位
     * @return 四舍五入后的结果
     */
    public static double round(double v,int scale){
        if(scale<0){
            throw new IllegalArgumentException(
                "The scale must be a positive integer or zero");
        }
        BigDecimal b = new BigDecimal(Double.toString(v));
        BigDecimal one = new BigDecimal("1");
        return b.divide(one,scale,BigDecimal.ROUND_HALF_UP).doubleValue();
    }
};
```


### BigDecimal 相关

#### java.math.BigDecimal 原理
不可变的、任意精度的有符号十进制数。BigDecimal 由任意精度的整数非标度值和32位的整数标度(scale)组成。
如果为零或正数，则标度是小数点后的位数。如果为负数，则将该数的非标度值乘以10的负scale次幂。
因此，BigDecimal表示的数值是(unscaledValue × 10-scale)。
与之相关的还有两个类：
java.math.MathContext：
该对象是封装上下文设置的不可变对象，它描述数字运算符的某些规则，如数据的精度，舍入方式等。
java.math.RoundingMode：
这是一种枚举类型，定义了很多常用的数据舍入方式。
这个类用起来还是很比较复杂的，原因在于舍入模式，数据运算规则太多太多，
不是数学专业出身的人看着中文API都难以理解，这些规则在实际中使用的时候在翻阅都来得及。
在银行、帐户、计费等领域，BigDecimal提供了精确的数值计算。其中8种舍入方式值得掌握。

#### BigDecimal 的8中舍入模式

```
1、ROUND_UP
舍入远离零的舍入模式。
在丢弃非零部分之前始终增加数字(始终对非零舍弃部分前面的数字加1)。
注意，此舍入模式始终不会减少计算值的大小。
2、ROUND_DOWN
接近零的舍入模式。
在丢弃某部分之前始终不增加数字(从不对舍弃部分前面的数字加1，即截短)。
注意，此舍入模式始终不会增加计算值的大小。
3、ROUND_CEILING
接近正无穷大的舍入模式。
如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同;
如果为负，则舍入行为与 ROUND_DOWN 相同。
注意，此舍入模式始终不会减少计算值。
4、ROUND_FLOOR
接近负无穷大的舍入模式。
如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;
如果为负，则舍入行为与 ROUND_UP 相同。
注意，此舍入模式始终不会增加计算值。
5、ROUND_HALF_UP
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。
如果舍弃部分 >= 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同。
注意，这是我们大多数人在小学时就学过的舍入模式(四舍五入)。
6、ROUND_HALF_DOWN
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。
如果舍弃部分 > 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同(五舍六入)。
7、ROUND_HALF_EVEN
向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。
如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND_HALF_UP 相同;
如果为偶数，则舍入行为与 ROUND_HALF_DOWN 相同。
注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。
此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。
如果前一位为奇数，则入位，否则舍去。
以下例子为保留小数点1位，那么这种舍入方式下的结果。
1.15>1.2 1.25>1.2
8、ROUND_UNNECESSARY
断言请求的操作具有精确的结果，因此不需要舍入。
如果对没有获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。
```
