# KMP算法

KMP主要思想：当出现字符串不匹配时，可以知道一部分已经匹配的内容，避免从头匹配。

## 题目

[28.找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/)

## 解法

**前缀表**

前缀表存放的是，前`i`个字符组成的前缀`s[0,i]`中，前缀和后缀最长相等的长度。

![202212131801592](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071603762.jpg)

```java
private void getNext(String str, int[] next) {
    next[0] = 0;
    int j = 0;//j为相等前缀的长度，即str[0,j)为相等前缀。
    for (int i = 1; i < str.length(); ++i) {
        while (j > 0 && str.charAt(i) != str.charAt(j)) j = next[j-1]; //注意是用while而不是if，循环回退，不仅仅回退一次。
        if (str.charAt(i) == str.charAt(j)) j++;
        next[i] = j;
    }
}
```



**匹配**

![202212131801597](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071603772.jpg)

当`i==6` `j==6`时，二者不相等，由于前缀表已经记录了最大相等的前后缀，已知`s2[0,j-1]`前缀的最大相等前后缀为`2`，将`j`回退到`next[j-1]`即可避免中间的无用匹配。

```java
public int strStr(String haystack, String needle) {
    int[] next = new int[needle.length()];
    getNext(needle, next);
    int j = 0;
    for(int i = 0; i < haystack.length(); ++i){
        while (j > 0 && haystack.charAt(i) != needle.charAt(j)) j = next[j-1];
        if (haystack.charAt(i) == needle.charAt(j)) {
            j++;
        }
        if (j == needle.length()) return i-needle.length()+1;
    }
    return -1;
}
```

## 参考资料

[代码随想录 - 力扣28题题解](https://programmercarl.com/0028.%E5%AE%9E%E7%8E%B0strStr.html)

[帮你把KMP算法学个通透！（理论篇）](https://www.bilibili.com/video/BV1PD4y1o7nd/)
