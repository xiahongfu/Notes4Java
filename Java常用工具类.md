# Java常用工具类
#### Arrays
- binarySearch：二分查找
- compare(a, b)：比较两数组字典序，正数代表a字典序>b字典序compareUnsigned是将数组元素处理成无符号数。
- copyOf、copyOfRange：复制数组
- deepToString：返回数组的“深度内容”。用于多维数组打印内容

```java
  public static void main(String[] args) {
      int a[] = {1, 2, 3};
      System.out.println(Arrays.toString(a));   //[1, 2, 3]
      int b[][] = {{1, 2, 3}, {4, 5, 6}}; 
      System.out.println(Arrays.toString(b)); //[[I@14ae5a5, [I@7f31245a]]
      System.out.println(Arrays.deepToString(b)); //[[1, 2, 3], [4, 5, 6]]
  }
```
- deepHashCode、deepEquals：同上，用于多维数组的比较和基于多维数组的“深度内容”生成哈希值。
- equals、hashCode：内容相等才等，基于内容生成哈希值。
- fill：将指定值赋给指定数组。
- misMatch：返回两数组第一个不匹配的值。若无，则返回-1。
- ==parallelPrefix：暂时还没研究明白。==
- ==setAll：暂时还没研究明白。==
- sort：使用的是Dual-Pivot Quicksort，即 使用两个元素来分割数组。通常比传统 (one-pivot) 快排算法要快。
- ==spliterator、stream：都没研究明白==
- toString：返回数组内容。
#### String
String类提供了很多有用的方法，基本上能想到的都提供了
- charAt(i)：返回第i个字符。
- codePointAt(i)：返回第i个字符的unicode码
- compareTo：比较字典序。还有个忽略大小写的比较字典序
- concat：字符串拼接
- startWith、endsWith：判断是否以指定字符串（开头）结尾
- formate：将字符串格式化
- indent(n)：调整行缩进
- indexOf、lastIndexOf：返回指定字符串第一次（最后一次）出现的位置
- isBlank：判断字符串中是否包含除空格外的其他值。
- matches：判断字符串是否与指定正则表达式匹配
- regionMatches，判断两字符串是否在给定区间内相等。
- repeat：返回该字符串重复拼接。
- replace：替换字符。
- split：分割字符串
- strip、trim、stripLeading、stripTrailing：去空格
- substring：返回子字符串
- toCharArray()：string转char数组
- toLowerCase、toUpperCase：转换大小写。
- valueOf：将传入的参数转换成string

#异常的继承结构
Throwable：Error、Exception
Error：不可处理的错误。
Exception：RuntimeException、其它子类。
RuntimeException：运行时异常，编译阶段可以处理也可以不处理，如空指针异常、类转换异常等。
Exception的其它子类：编译时异常。在编译阶段必须进行处理，否则会报错。也就是说，在编写程序阶段必须对这种异常进行处理。

在自定义异常类时，可以根据需求继承Exception或者RuntimeException。