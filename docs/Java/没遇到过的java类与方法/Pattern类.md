# Pattern类

**声明：**public final class Pattern  implements java.io.Serializable

Pattern类有final 修饰，可知他不能被子类继承。

**含义：**模式类，正则表达式的编译表示形式。

**注意：**此类的实例是不可变的，可供多个并发线程安全使用。


Pattern Pattern.complie(String regex,int flag)，它接受一个标记参数flag，以调整匹配的行为。
flag来自以下Pattern类中的常量：

|常量|作用|
|--|--|
Pattern.CANON_EQ	|两个字符当且仅当它们的完全规范分解相匹配时，就认为它们是匹配的，例如，如果我们指定这个标记，表达式a\u030A就会匹配字符串？。在默认的情况下，匹配不考虑规范的等价性
Pattern.CASE_INSENSITIVE(?i)|	默认情况下，大小写不敏感的匹配假定只有US-ASCII字符集中的字符才能进行。这个标记允许模式匹配不必考虑大小写（大写或小写）。通过指定UNICODE_CASE标记及结合此标记，基于Unicode的大小写不敏感的匹配就可以开启了,也可以使用嵌入的标记表达式?i开启，下同
Pattern.COMMENTS(?x)	|在这种模式下，表达式中的空格(不是指\s,单纯指空格)将被忽略掉，并且以#开始直到行末的注释也会被忽略掉。通过嵌入的标记表达式也可以开启Unix的行模式
Pattern.DOTALL(?s)	|在dotall模式中，表达式“.”匹配所有字符，包括行终结符。默认情况下，“.”表达式不匹配行终结符
Pattern.MULTLINE(?m)|	在多行模式下，表达式^和$分别匹配一行或输入字符串的开始和结束。默认情况下，这些表达式仅匹配输入的完整字符串的开始和结束
Pattern.UNICODE_CASE(?u)	|当指定这个标记，并且开启CASE_INSENSITIVE时，大小写不敏感的匹配将按照与Unicode标准相一致的方式进行。默认情况下，大小写不敏感的匹配假定只能在US-ASCII字符集中的字符才能进行
Pattern.UNIX_LINES(?d)|	在这种模式下，在.、^和$行为中，只识别行终结符\n
