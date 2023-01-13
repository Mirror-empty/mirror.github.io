# StringTokenizer类

作用：用户分隔字符串

构造方法

构造一个用来解析 str 的 StringTokenizer 对象。java 默认的分隔符是空格("")、制表符(\t)、换行符(\n)、回车符(\r)。
```java
 public StringTokenizer(String str, String delim, boolean returnDelims) {
        currentPosition = 0;
        newPosition = -1;
        delimsChanged = false;
        this.str = str;
        maxPosition = str.length();
        delimiters = delim;
        retDelims = returnDelims;
        setMaxDelimCodePoint();
    }

    /**

     * @param   str     a string to be parsed.
     * @param   delim   the delimiters.
     * @exception NullPointerException if str is <CODE>null</CODE>
     */
    public StringTokenizer(String str, String delim) {
        this(str, delim, false);
    }


    public StringTokenizer(String str) {
        this(str, " \t\n\r\f", false);
    }
```

常用方法：

1. int countTokens()：返回nextToken方法被调用的次数。
2. boolean hasMoreTokens()：返回是否还有分隔符。
3. boolean hasMoreElements()：判断枚举 （Enumeration） 对象中是否还有数据。
4. String nextToken()：返回从当前位置到下一个分隔符的字符串。
5. Object nextElement()：返回枚举 （Enumeration） 对象的下一个元素。
6. String nextToken(String delim)：与 4 类似，以指定的分隔符返回结果。