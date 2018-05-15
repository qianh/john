---
title: java源代码赏析 —— String类
date: 2018-05-03 13:17:21
tags: java 源码
categories: java
---
#### (1) String类声明
##### String类实现了序列化、比较器两个接口，用final修饰符说明该类不能被其他类继承，其他类不能重写该类已有的方法
``` java
package java.lang; //定义在java.lang包下

public final class String implements java.io.Serializable, Comparable<String>, CharSequece
```
#### (2) 字符串的本质是 char[]
``` java 
/** The value is used for character storage. */ //该数组用于字符存储
private final char value[]; //String 类底层是 char 数组
```
#### (3) String 类的无参构造方法
``` java
public String {
  this.value = new char[0]; //构造一个空串
}
```
#### (4) String 类的其他参构造方法
##### 4.1 String类型
``` java
public String(String original) {
  this.value = original.value;
  this.hash = original.hash;
}
```
##### 4.2 使用 char[] 构造一个字符串
###### 调用 Arrays 类字符串拷贝函数复制一份字符完成构造
``` java
public String(Char value[]) {
  this.value = Arrays.copyOf(value, value.length);
}
public String(Char value[], int offset, int count) {
  //处理非法参数
  if(offset < 0) {
    throw new StringIndexOutOfBoundsException(offset);
  }
  if(count < 0) {
    throw new StringIndexOutOfBoundsException(count);
  }
  //Note: offset or count might be near -1>>>1.
  if(offset > value.length - count) {
    throw new StringIndexOutOfBoundsException(offset + count);
  }
  //参数合法，调用字符串复制函数复制目标char[]到value[]
  this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```
##### 4.3 int[] 构造一个字符串
``` java
public String(int[] codePoints, int offset, int count) {
  if (offset < 0) {
    throw new StringIndexOutOfBoundsException(offset);
  }
  if (count < 0) {
    throw new StringIndexOutOfBoundsException(count);
  }
  // Note: offset or count might be near -1>>>1.
  if (offset > codePoints.length - count) {
    throw new StringIndexOutOfBoundsException(offset + count);
  }

  final int end = offset + count;

  // Pass 1: Compute precise size of char[]
  int n = count;
  for (int i = offset; i < end; i++) {
    int c = codePoints[i];
    if (Character.isBmpCodePoint(c))
      continue;
    else if (Character.isValidCodePoint(c))
      n++;
    else throw new IllegalArgumentException(Integer.toString(c));
  }

  // Pass 2: Allocate and fill in char[]
  final char[] v = new char[n];

  for (int i = offset, j = 0; i < end; i++, j++) {
    int c = codePoints[i];
    if (Character.isBmpCodePoint(c))
      v[j] = (char)c;
    else
      Character.toSurrogates(c, v, j++);
  }

  this.value = v;
}
```
##### 4.4 bytes[] 构造一个字符串
``` java
public String(byte bytes[]) {
  this(bytes, 0, bytes.length);
}

public String(byte bytes[], int offset, int length) {
  //offset 是起始位置，length是从起始位置计要构造字符串的长度
  checkBounds(bytes, offset, length);
  this.value = StringCoding.decode(bytes, offset, length);
}

public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
  if (charsetName == null)
      throw new NullPointerException("charsetName");
  checkBounds(bytes, offset, length);
  this.value = StringCoding.decode(charsetName, bytes, offset, length);
}

public String(byte bytes[], int offset, int length, Charset charset) {
  if (charset == null)
      throw new NullPointerException("charset");
  checkBounds(bytes, offset, length);
  this.value =  StringCoding.decode(charset, bytes, offset, length);
}

public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
  this(bytes, 0, bytes.length, charsetName);
}

public String(byte bytes[], Charset charset) {
  this(bytes, 0, bytes.length, charset);
}
```
##### 4.5 StringBuffer（线程安全） 构造一个字符串
``` java 
public String(StringBuffer buffer) {
  synchronized(buffer) {
    this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
  }
}
```
##### 4.6 StringBuilder（线程不安全） 构造一个字符串
``` java 
public String(StringBuilder builder) {
  this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```
#### (5) 检查边界的函数
##### 如果越界会抛出异常 StringIndexOutOfBoundsException
``` java
private static void checkBounds(byte[] bytes, int offset, int length) {
  if(length < 0) {
    throw new StringIndexOutOfBoundsException(length);
  }
  if(offset < 0) {
    throw new StringIndexOutOfBoundsException(offset);
  } 
  if(offset > bytes.length - length) {
    throw new StringIndexOutOfBoundsException(offset + length);
  }
}
```
#### (6) 获取字符串的长度
##### 字符串的长度也就是value数组的长度，即有效字的个数。
``` java
public int length() {
  return value.length;
}
```
#### (7) 判断字符串是否是空串
``` java
public boolean isEmpty() {
  return value.length == 0; //判断其有效字符个数是否为0
}
```
#### (8) 获取某一位置的字符
##### 如果传入的索引合法，就返回该位置的字符
``` java
public char charAt(int index) {
  if((index < 0) || (index >= value.length)) {
    throw new StringIndexOutOfBoundsException(index);
  }
  return value[index];
}
```
#### (9) 获取某一位置的字符对应的 int 值
``` java
public int codePointAt(int index) {
  if ((index < 0) || (index >= value.length)) {
    throw new StringIndexOutOfBoundsException(index);
  }
  return Character.codePointAtImpl(value, index, value.length);
}
```
#### (10) 将字符串转换为 byte[] 数组
``` java
public byte[] getBytes() {
  return StringCoding.encode(value, 0, value.length);
}
```
#### (11) <font color="#dd0000"> (重要) </font> 字符串比较函数 equals()
##### 不同于 "==" 比较，该函数比较的是字符串的内容而非引用，前者比较的是引用（地址）
``` java 
public boolean equals(Object anObject) { //使用Object类接收，向上转型
  if(this == anObject) {
    return true;
  }
  if(anObject instanceof String) { //确定是否是String类的对象
    String anotherString = (String)anObject;//强制转换为String
    int n = value.length;
    //先比较长度,再比较内容
    if(n == anotherString.value.length) {
      char v1[] = value;
      char v2[] = anotherString.value;
      int i = 0;
      while(n-- != 0) {
        if(v1[i] != v2[i]) {
          return false;
          i++;
        }
      }
      return true;
    }
  }
  return false; //不是String类型，返回false
}
```
#### (12) 比较两个字符串内容，忽略大小写
``` java
public boolean equalsIgnoreCase(String anotherString) {
  return (this == anotherString) ? true : (anotherString != null)
          && (anotherString.value.length == value.length)
          && regionMatches(true, 0, anotherString, 0, value.length);
}

public boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len) {
  char ta[] = value;
  int to = toffset;
  char pa[] = other.value;
  int po = ooffset;
  // Note: toffset, ooffset, or len might be near -1>>>1.
  if ((ooffset < 0) || (toffset < 0)
        || (toffset > (long)value.length - len)
        || (ooffset > (long)other.value.length - len)) {
    return false;
  }
  while (len-- > 0) {
    char c1 = ta[to++];
    char c2 = pa[po++];
    if (c1 == c2) {
        continue;
    }
    if (ignoreCase) {
      // If characters don't match but case may be ignored,
      // try converting both characters to uppercase.
      // If the results match, then the comparison scan should
      // continue.
      // 先转大写进行比较
      char u1 = Character.toUpperCase(c1);
      char u2 = Character.toUpperCase(c2);
      if (u1 == u2) {
        continue;
      }
      // Unfortunately, conversion to uppercase does not work properly
      // for the Georgian alphabet, which has strange rules about case
      // conversion.  So we need to make one last check before
      // exiting.
      // 如果转大写比较不成功（由于某种原因），转小写再进行比较一次
      if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
        continue;
      }
    }
    return false;
  }
  return true;
}
```
#### (13) 判断该字符串是否以某个指定字符串开头
``` java
public boolean startsWith(String prefix, int toffset) {
  char ta[] = value;
  int to = toffset;
  char pa[] = prefix.value;
  int po = 0;
  int pc = prefix.value.length;
  // Note: toffset might be near -1>>>1.
  if ((toffset < 0) || (toffset > value.length - pc)) {
    return false;
  }
  while (--pc >= 0) {
    if (ta[to++] != pa[po++]) {
      return false;
    }
  }
  return true;
}

public boolean startsWith(String prefix) {
  return startsWith(prefix, 0);
}
```
#### (14) 判断该字符串是否以某个指定字符串结尾
``` java
public boolean endsWith(String suffix) {
  return startsWith(suffix, value.length - suffix.value.length);
}
```
#### (15) hashCode 生成规则
``` java
public int hashCode() {
  int h = hash;
  if (h == 0 && value.length > 0) {// 没有生成过的时候才进入
    char val[] = value;

    for (int i = 0; i < value.length; i++) {
        h = 31 * h + val[i];
    }
    hash = h;
  }
  return h;
}
```
#### (16) 查找字符串的位置
``` java
public int indexOf(String str) {
  return indexOf(str, 0);
}

public int indexOf(String str, int fromIndex) {
  return indexOf(value, 0, value.length,
          str.value, 0, str.value.length, fromIndex);
}

static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
  //source 原值， target 查找目标值， offset 起始位置， count 字符串长度 
  //fromIndex 开始查找的位置
  if (fromIndex >= sourceCount) {
    return (targetCount == 0 ? sourceCount : -1);
  }
  if (fromIndex < 0) {
    fromIndex = 0;
  }
  if (targetCount == 0) {
    return fromIndex;
  }

  char first = target[targetOffset];
  int max = sourceOffset + (sourceCount - targetCount);

  for (int i = sourceOffset + fromIndex; i <= max; i++) {
    /* Look for first character. */
    if (source[i] != first) {
      while (++i <= max && source[i] != first);
    }

    /* Found first character, now look at the rest of v2 */
    if (i <= max) {
      int j = i + 1;
      int end = j + targetCount - 1;
      for (int k = targetOffset + 1; j < end && source[j]
              == target[k]; j++, k++);

      if (j == end) {
        /* Found whole string. */
        return i - sourceOffset;
      }
    }
  }
  return -1;
}
```
#### (17) 查找字符串的最后位置
``` java
public int lastIndexOf(String str) {
  return lastIndexOf(str, value.length);
}
public int lastIndexOf(String str, int fromIndex) {
  return lastIndexOf(value, 0, value.length,
          str.value, 0, str.value.length, fromIndex);
}
static int lastIndexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
  /*
    * Check arguments; return immediately where possible. For
    * consistency, don't check for null str.
    */
  int rightIndex = sourceCount - targetCount;
  if (fromIndex < 0) {
      return -1;
  }
  if (fromIndex > rightIndex) {
      fromIndex = rightIndex;
  }
  /* Empty string always matches. */
  if (targetCount == 0) {
      return fromIndex;
  }

  int strLastIndex = targetOffset + targetCount - 1;
  char strLastChar = target[strLastIndex];
  int min = sourceOffset + targetCount - 1;
  int i = min + fromIndex;

startSearchForLastChar:
  while (true) {
    while (i >= min && source[i] != strLastChar) {
      i--;
    }
    if (i < min) {
      return -1;
    }
    int j = i - 1;
    int start = j - (targetCount - 1);
    int k = strLastIndex - 1;

    while (j > start) {
      if (source[j--] != target[k--]) {
        i--;
        continue startSearchForLastChar;
      }
    }
    return start - sourceOffset + 1;
  }
}
```
#### (18) 截取子串
``` java
public String substring(int beginIndex) {
  if (beginIndex < 0) {
    throw new StringIndexOutOfBoundsException(beginIndex);
  }
  int subLen = value.length - beginIndex;
  if (subLen < 0) {
    throw new StringIndexOutOfBoundsException(subLen);
  }
  return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}

public String substring(int beginIndex, int endIndex) {
  if (beginIndex < 0) {
    throw new StringIndexOutOfBoundsException(beginIndex);
  }
  if (endIndex > value.length) {
    throw new StringIndexOutOfBoundsException(endIndex);
  }
  int subLen = endIndex - beginIndex;
  if (subLen < 0) {
    throw new StringIndexOutOfBoundsException(subLen);
  }
  return ((beginIndex == 0) && (endIndex == value.length)) ? this
      : new String(value, beginIndex, subLen);
}
```
#### (19) 字符串连接
``` java
public String concat(String str) {
  int otherLen = str.length();
  if (otherLen == 0) {
    return this;
  }
  int len = value.length;
  char buf[] = Arrays.copyOf(value, len + otherLen);
  str.getChars(buf, len);
  return new String(buf, true);
}

void getChars(char dst[], int dstBegin) {
  System.arraycopy(value, 0, dst, dstBegin, value.length);
}

String(char[] value, boolean share) {
  // assert share : "unshared not supported";
  this.value = value;
}
```
#### (20) 替换全部
##### 替换原字符串中符合要求的所有片段
``` java
public String replace(char oldChar, char newChar) {
  if (oldChar != newChar) {
    int len = value.length;
    int i = -1;
    char[] val = value; /* avoid getfield opcode */

    while (++i < len) {
      if (val[i] == oldChar) {
        break;
      }
    }
    if (i < len) {
      char buf[] = new char[len];
      for (int j = 0; j < i; j++) {
        buf[j] = val[j];
      }
      while (i < len) {
        char c = val[i];
        buf[i] = (c == oldChar) ? newChar : c;
        i++;
      }
      return new String(buf, true);
    }
  }
  return this;
}

public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
```
#### (21) 是否匹配该正则表达式
``` java 
public boolean matches(String regex) {
  return Pattern.matches(regex, this);
}
```
#### (22) 是否包含该字符
``` java
public boolean contains(CharSequence s) {
  return indexOf(s.toString()) > -1;
}
```
#### (23) 字符串拆分
``` java 
public String[] split(String regex) {
  return split(regex, 0);
}

public String[] split(String regex, int limit) {
  /* fastpath if the regex is a
    (1)one-char String and this character is not one of the
      RegEx's meta characters ".$|()[{^?*+\\", or
    (2)two-char String and the first char is the backslash and
      the second is not the ascii digit or ascii letter.
    */
  char ch = 0;
  if (((regex.value.length == 1 &&
        ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
        (regex.length() == 2 &&
        regex.charAt(0) == '\\' &&
        (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
        ((ch-'a')|('z'-ch)) < 0 &&
        ((ch-'A')|('Z'-ch)) < 0)) &&
      (ch < Character.MIN_HIGH_SURROGATE ||
        ch > Character.MAX_LOW_SURROGATE))
  {
      int off = 0;
      int next = 0;
      boolean limited = limit > 0;
      ArrayList<String> list = new ArrayList<>();
      while ((next = indexOf(ch, off)) != -1) {
          if (!limited || list.size() < limit - 1) {
              list.add(substring(off, next));
              off = next + 1;
          } else {    // last one
              //assert (list.size() == limit - 1);
              list.add(substring(off, value.length));
              off = value.length;
              break;
          }
      }
      // If no match was found, return this
      if (off == 0)
          return new String[]{this};

      // Add remaining segment
      if (!limited || list.size() < limit)
          list.add(substring(off, value.length));

      // Construct result
      int resultSize = list.size();
      if (limit == 0) {
          while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
              resultSize--;
          }
      }
      String[] result = new String[resultSize];
      return list.subList(0, resultSize).toArray(result);
  }
  return Pattern.compile(regex).split(this, limit);
}
```
#### (24) 去除两端空格
``` java 
public String trim() {
  int len = value.length;
  int st = 0;
  char[] val = value;    /* avoid getfield opcode */

  while ((st < len) && (val[st] <= ' ')) {
    st++;
  }
  while ((st < len) && (val[len - 1] <= ' ')) {
    len--;
  }
  return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```
#### (25) toString 方法
``` java
public String toString() {
  return this;
}
```