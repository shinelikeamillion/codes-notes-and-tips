Java中保留两位小数，并四舍五入
=============================

在Java中，有的时候针对浮点数的处理时，需要保留两位小数，第一时间想到的是工具类DecimalFormat。可以使用如下方式：
> DecimalFormat df  = new DecimalFormat("#0.00");
> df.format(srcNumber);

其实还有一种更加简单的方式（这种方式会对指定的浮点数进行四舍五入）：
```
public static Double keepTwoDecimalPlaces(Double srcNumber){
    String stringNumber = parseToTwoDecimalPlacesString(srcNumber);
    return StringUtils.isEmpty(stringNumber)?null:Double.parseDouble(stringNumber);
}
public static String parseToTwoDecimalPlacesString(Double srcNumber){
    return srcNumber==null?null: String.format("%.2f", srcNumber);
}
```
可用以下代码验证结果：
```
public static void main(String[] args) {
    System.out.println(String.format("%.10f",Math.PI));
}
```
原理及用法

String.format 作为文本处理工具，为我们提供强大而丰富的字符串格式化功能。

对浮点数进行格式化　　　　　　　　　　　　　　　　　　　　　　　　

占位符格式为： %[index$][标识]*[最小宽度][.精度]转换符
```
double num = 123.4567899;
System.out.print(String.format("%f %n", num)); // 123.456790 
System.out.print(String.format("%a %n", num)); // 0x1.edd3c0bb46929p6 
System.out.print(String.format("%g %n", num)); // 123.457
```
可用标识：

* -，在最小宽度内左对齐,不可以与0标识一起使用。
* 0，若内容长度不足最小宽度，则在左边用0来填充。
* #，对8进制和16进制，8进制前添加一个0,16进制前添加0x。
* +，结果总包含一个+或-号。
* 空格，正数前加空格，负数前加-号。
* ,，只用与十进制，每3位数字间用,分隔。
* (，若结果为负数，则用括号括住，且不显示符号。
* 可用转换符：

* b，布尔类型，只要实参为非false的布尔类型，均格式化为字符串true，否则为字符串false。
* n，平台独立的换行符, 也可通过System.getProperty("line.separator")获取。
* f，浮点数型（十进制）。显示9位有效数字，且会进行四舍五入。如99.99。
* a，浮点数型（十六进制）。
* e，指数类型。如9.38e+5。
* g，浮点数型（比%f，%a长度短些，显示6位有效数字，且会进行四舍五入）
