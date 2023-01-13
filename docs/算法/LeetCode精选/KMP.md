# KMP

KMP 最难的是next数组的构造，而next数组构造是基于子字符串自己去匹配自己的前缀和后缀的，前缀是从开头到结尾，后缀反过来的。

```java
 public static int strStr(String haystack, String needle) {

        haystack.indexOf(needle);
    //    先找出kmp
        // 求next数组：next[i]表示needle[i]前面最长公共前缀索引
        int [] next= new int[needle.length()];
        char[] s = haystack.toCharArray();
        char[] p = needle.toCharArray();
        next[0] = -1;
        // j为 最长最近结尾索引 i为待匹配的字符，j+1的字符就是要于位置i去匹配的字符 例如：字符串ababc，开始i为1，j为-1
        for (int i = 1,j=-1;i <needle.length() ; i++) {
            // p[i] 为待匹配字符 ，p[j+1]为最长前缀后一个字符
            while (j>=0&&p[i]!=p[j+1]){
                j=next[j];
            }
            //如果匹配上了 j++
            if (p[i]==p[j+1]){
                j++;
            }
            next[i]=j;
        }

        for (int i = 0,j=-1; i < s.length; i++) {
            while (j>=0&&s[i]!=p[j+1]){
                // 没有匹配上，往后面走
                j=next[j];
            }
            if (s[i]==p[j+1]){
                j++;
            }
            if (j==p.length-1){
                return i-p.length+1;
            }
        }
        return -1;

    }
```

