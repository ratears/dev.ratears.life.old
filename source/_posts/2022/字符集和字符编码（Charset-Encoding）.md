---
title: 字符集和字符编码（Charset & Encoding）
date: 2022-09-10 21:03:57
auth: ratears
update: 2022-08-31 02:12:50
categories:
	- [computer,character encoding]
tags:
	- computer
	- character encoding
---

# 引言

- 复习Java I/O知识的时候，关于转换流有涉及到字符集的知识。于是想深入理解一下字符集和编码。（毕竟，字符编码是计算机技术、编程的基石，包括编程中涉及到的国际化问题，就必须懂得一点字符编码的知识）查阅相关资料后，并做了笔记

<br>

```java
// charsetName：指定字符编码 
public InputStreamReader(InputStream in, String charsetName)
public OutputStreamWriter(OutputStream out, String charsetName)
```

```java
InputStreamReader isr = new InputStreamReader(fis, "GBK");
OutputStreamWriter osw = new OutputStreamWriter(fos, "GBK");
```

<br>

<br>

<br>

# 基础知识

## 编码和解码

- 编码：信息按照一定的规则从一种形式或格式转换为另一种形式的过程
- 解码：是一个编码的逆转换过程

> <font color="red">编码解码都是有一套预先规定的方案,无论是在编码过程还是解码过程,都要遵守这套规则来运算</font>

<br>

<br>

## 为什要编码解码

- 在计算机中，是不能直接存储字符的，计算机中储存的信息都是用二进制数表示的；而我们在屏幕上看到的英文、汉字等字符是二进制数转换之后的结果。通俗的说，按照何种规则将字符存储在计算机中，如'a'用什么表示，称为"编码"；反之，将存储在计算机中的二进制数解析显示出来，称为"解码"，如同密码学中的加密和解密。在解码过程中，如果使用了错误的解码规则，则导致'a'解析成'b'或者**乱码**
- 编码解码在不同的场景中具有不同的意义，比如常见的字符编码解码,URL编码解码等

> 计算机为什么采用二进制：有很多原因.在这里简单的说几点,在技术上易实现,因为我们可以使用双稳态电路来表示1和0,高电平为1,低电平为0.且因为只有1和0,所以在传输和处理的过程中不容易出错.另外二进制的运算规则也相对来说简单.综合各方面因素,最终计算机采用二进制

<br>

<br>

## 字符集（Charset）

- 字符集（Charset）是一个系统支持的所有抽象字符的集合。字符是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等

<br>

<br>

## 字符编码（Character Encoding）

- 字符编码（Character Encoding）是一套法则，使用该法则能够对自然语言的字符的一个集合（如字母表或音节表），与其他东西的一个集合（如号码或电脉冲）进行配对。即在符号集合与数字系统之间建立对应关系，它是信息处理的一项基本技术。通常人们用符号集合（一般情况下就是文字）来表达信息。而以计算机为基础的信息处理系统则是利用元件（硬件）不同状态的组合来存储和处理信息的。元件不同状态的组合能代表数字系统的数字，因此字符编码就是将符号转换为计算机可以接受的数字系统的数，称为数字代码（通俗的讲：计算机字符编码就是将 计算机里展示的字符集 和 计算机能理解的二进制数 对应起来）

<br>

<br>

## 乱码

- 乱码就是因为使用了不对应的字符集导致部分或所有字符没法被正确阅读。（就比如我告诉你,打开新华字典的xx页的xx行,就是我想对你说的话.结果你拿了本牛津英汉字典,那你当然不能正确获取到我想对你说的话）

<br>

<br>

### 为什么有时候乱码都是 ? 号

- 在 `Java` 开发中，经常会碰到乱码显示为 `?` 号，比如下面这个例子：

```java
String name = "双子孤狼";
byte[] bytes = name.getBytes(StandardCharsets.ISO_8859_1);
System.out.println(new String(bytes));//输出：????
```

- 这个输出结果的原因是中文无法用 `ISO_8859_1` 编码进行存储，而示例中却强制用 `ISO_8859_1` 编码进行解码

> 在 `Java` 中提供了一个 `ISO_8859_1` 类用来解码，解码时当发现当前字符转成十进制之后大于 `255` 时就会直接不进行解码，转而直接赋一个默认值 `63`，所以上面的示例中的 `byte` 数组结果就是 `63 63 63 63`，而`63` 在 `ASCII` 中就恰好就对应了 `?` 号。
>
> 所以一般我们看到编码出现 `?` 基本就说明当前是采用 `ISO_8859_1` 进行的解码，而当前的字符又大于 `255`

<br>

<br>

<br>

# 常用字符集和字符编码

- 常见字符集名称：ASCII字符集、GB2312字符集、BIG5字符集、GB18030字符集、Unicode字符集等。计算机要准确的处理各种字符集文字，需要进行字符编码，以便计算机能够识别和存储各种文字

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/微信图片_20220910225027-(1).2kjjnk2mwyy0.webp" width="70%"/>

<br>

<br>

<br>

## ASCII字符集&编码

- 计算机最开始诞生于美国，而且计算机只能识别二进制，所以我们就需要把常用语言和二进制关联起来。美国人把英文里面常用的字符以及一些控制字符转换成了二进制数据，形成了一个编码对应关系表，这就是ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）字符集和ASCII编码
- ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统。它主要用于显示现代英语，而其扩展版本EASCII则可以勉强显示其他西欧语言。它是现今最通用的单字节编码系统（但是有被Unicode追上的迹象），并等同于国际标准ISO/IEC 646

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6hglwgjozx80.webp" width="70%"/>

<br>

- **ASCII字符集：**主要包括控制字符（回车键、退格、换行键等）；可显示字符（英文大小写字符、阿拉伯数字和西文符号）
- **ASCII编码：**将ASCII字符集转换为计算机可以接受的数字系统的数的规则

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3qu3exerer80.webp" width="60%"/>

<br>

> 使用7位（bits）表示一个字符，共128字符；但是7位编码的字符集只能支持128个字符，为了表示更多的欧洲常用字符对ASCII进行了扩展，ASCII扩展字符集使用8位（bits）表示一个字符，共256字符

<br>

- ASCII编码表

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.632akue1h1s0.webp" width="70%"/>

<br>

- 扩展ASCII编码表

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4selgy9t3da0.webp" width="70%"/>

<br>

> <font color="red">ASCII的最大缺点是只能显示26个基本拉丁字母、阿拉伯数目字和英式标点符号，因此只能用于显示现代美国英语（而且在处理英语当中的外来词如naïve、café、élite等等时，所有重音符号都不得不去掉，即使这样做会违反拼写规则）。而EASCII虽然解决了部份西欧语言的显示问题，但对更多其他语言依然无能为力。因此现在的苹果电脑已经抛弃ASCII而转用Unicode</font>

<br>

<br>

## GBXXXX字符集&编码

- 当天朝也有了计算机之后，ASCII字符集（编码）不够用。为了显示中文，必须设计一套编码规则用于将汉字转换为计算机可以接受的数字系统的数
- 天朝专家把那些127号之后的奇异符号们（即EASCII）取消掉，规定：一个小于127的字符的意义与原来相同，但两个大于127的字符连在一起时，就表示一个汉字，前面的一个字节（他称之为高字节）从0xA1用到 0xF7，后面一个字节（低字节）从0xA1到0xFE，这样我们就可以组合出大约7000多个简体汉字了。在这些编码里，还把数学符号、罗马希腊的 字母、日文的假名们都编进去了，连在ASCII里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的"全角"字符，而原来在127号以下的那些就叫"半角"字符了

<br>

<br>

### GB2312字符集&编码

#### GB2312字符集设计

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.376tsyyf9wu0.webp" width="70%"/>

<br>

- 01~09区表示除汉子外的682个字符

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3x9vnc4ax1c0.webp" width="70%"/>

<br>

- 10~15区是空白区

- 16~55区收录3755个一级汉字，按拼音排序

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1g8kw6mqnk80.webp" width="70%"/>

<br>

- 56~87区收录了3008个二级汉字，按部首/笔画排序（不太常用的、生僻汉字）

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.x9xgkbmfbhc.webp" width="70%"/>

<br>

<br>

#### GB2312字符编码设计

- 规定如何在计算机中存储

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5lyrzte8k8o.webp" width="70%"/>

<br>

> 以`侃`这个字为例，`侃`这个字的码位是 `5709`，按前两位和后两位分开（`57`和`09`）并将其转化为16进制（`0x39`和`0x09`），再分别加上`0xA0`，得到`0xD9`和`0xA9`，再将其组合到一起得到`0xD90xA9`，`0xD90xA9`便是`侃`这个字的GB2312编码值。即计算机使用GB2312编码时，`侃`这个字在计算机中存储的16进制形式是：`D9A9`

- Java获取某个汉字的GB2312编码值

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.46qrewk6ons0.webp" width="70%"/>

<br>

<br>

### GB2312字符集扩展

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.585csd88dpw0.webp" width="70%"/>

<br>

<br>

<br>

## Unicode字符集&UTF编码

- Unicode 只是一个字符集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储

- unicode是一个标准，其包含了对应的字符集和编码规则。UTF-32/ UTF-16/ UTF-8是三种字符编码方案（即：UTF-8 是 Unicode 的实现方式之一）

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6film7n2p0s0.webp" width="70%"/>

<br>

<br>

### UTF-8 编码

- UTF-8（8-bit Unicode Transformation Format）是一种针对Unicode的**可变长度字符编码（定长码）**，也是一种前缀码。它可以用来表示Unicode标准中的任何字符，且其编码中的第一个字节仍与ASCII兼容，这使得原来处理ASCII字符的软件无须或只须做少部份修改，即可继续使用

- UTF-8编码是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度（每次传送8位数据）
- UTF-8 的编码规则（存储规范）：
  - 对于单字节的符号，字节的第一位设为`0`，后面7位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的
  - 对于`n`字节的符号（`n > 1`），第一个字节的前`n`位都设为`1`，第`n + 1`位设为`0`，后面字节的前两位一律设为`10`。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码

| Unicode符号范围（十六进制） | UTF-8编码方式（二进制）             |
| :-------------------------: | :---------------------------------- |
|  0x0000 0000 - 0x0000 007F  | 0xxxxxxx                            |
|  0x0000 0080 - 0x0000 07FF  | 110xxxxx 10xxxxxx                   |
|  0x0000 0800 - 0x0000 FFFF  | 1110xxxx 10xxxxxx 10xxxxxx          |
|  0x0001 0000 - 0x0010 FFFF  | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

> 跟据上表（字母x表示可用编码的位），解读 UTF-8 编码非常简单。如果一个字节的第一位是`0`，则这个字节单独就是一个字符；如果第一位是`1`，则连续有多少个`1`，就表示当前字符占用多少个字节

- 以汉字`严`为例，演示如何实现 UTF-8 编码

> `严`的 Unicode 是`4E25`（`100111000100101`），根据上表，可以发现`4E25`处在第三行的范围内（`0000 0800 - 0000 FFFF`），因此`严`的 UTF-8 编码需要三个字节，即格式是`1110xxxx 10xxxxxx 10xxxxxx`。然后，从`严`的最后一个二进制位开始，依次从后向前填入格式中的`x`，多出的位补`0`。这样就得到了，`严`的 UTF-8 编码是`11100100 10111000 10100101`，转换成十六进制就是`E4B8A5`。（即：计算机使用`Unicode`编码时，`严`这个字在计算机中存储的16进制形式是：`E4B8A5`）

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2hiq48s9b7k.webp" width="70%"/>

<br>

# 参考与延伸

- 阮一峰博客：[字符编码笔记：ASCII，Unicode 和 UTF-8](https://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
- runoob（菜鸟教程）：[字符集和字符编码](https://www.runoob.com/w3cnote/charset-encoding.html)
- [字符集历史](https://www.cnblogs.com/lonely-wolf/p/14676335.html)

<br>