---
title: isEmpty和isBlank的用法区别居然一半的人答不上来
date: 2024-06-29 22:18:46
tags: 字符串比较
categories: 面试
---



也许你两个都不知道,也许你除了isEmpty/isNotEmpty/isNotBlank/isBlank外,并不知道还有`isAnyEmpty/isNoneEmpty/isAnyBlank/isNoneBlank`的存在, come on ,让我们一起来探索`org.apache.commons.lang3.StringUtils;`这个工具类.

# isEmpty系列

StringUtils.isEmpty()
---------------------

`>>>`是否为空. 可以看到 `" "` 空格是会绕过这种空判断,因为是一个`空格`,并不是严格的`空值`,会导致 `isEmpty(" ")=false`

*   StringUtils.isEmpty(null) = true

*   StringUtils.isEmpty("") = true

*   StringUtils.isEmpty(" ") = false

*   StringUtils.isEmpty(“bob”) = false

*   StringUtils.isEmpty(" bob ") = false


```
/**  
  *  
  * <p>NOTE: This method changed in Lang version 2.0.  
  * It no longer trims the CharSequence.  
  * That functionality is available in isBlank().</p>  
  *  
  * @param cs  the CharSequence to check, may be null  
  * @return {@code true} if the CharSequence is empty or null  
  * @since 3.0 Changed signature from isEmpty(String) to isEmpty(CharSequence)  
*/  
public static boolean isEmpty(final CharSequence cs) {  
   return cs == null || cs.length() == 0;  
}  
  

```

StringUtils.isNotEmpty()
------------------------

`>>>`相当于不为空 , = `!isEmpty()`

```
public static boolean isNotEmpty(final CharSequence cs) {  
    return !isEmpty(cs);  
}  
  

```

StringUtils.isAnyEmpty()
------------------------

`>>>`是否有一个为空,`只有一个为空,就为true`.

*   StringUtils.isAnyEmpty(null) = true

*   StringUtils.isAnyEmpty(null, “foo”) = true

*   StringUtils.isAnyEmpty("", “bar”) = true

*   StringUtils.isAnyEmpty(“bob”, “”) = true

*   StringUtils.isAnyEmpty(" bob ", null) = true

*   StringUtils.isAnyEmpty(" ", “bar”) = false

*   StringUtils.isAnyEmpty(“foo”, “bar”) = false


![](./2024/06/29/isEmpty和isBlank的用法区别居然一半的人答不上来/1.png)

StringUtils.isNoneEmpty()
-------------------------

`>>>`相当于`!isAnyEmpty(css)` , 必须所有的值都不为空才返回true

[](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)如果你近期准备面试跳槽，建议在ddkk.com在线刷题，涵盖 一万+ 道 Java 面试题，几乎覆盖了所有主流技术面试题，还有市面上最全的技术五百套，精品系列教程，免费提供。[](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247500365&idx=1&sn=7abfd8b30427d4ac15bb351c3dbb640c&scene=21#wechat_redirect)

```
/**  
 * <p>Checks if none of the CharSequences are empty ("") or null.</p>  
 *  
 * <pre>  
 * StringUtils.isNoneEmpty(null)             = false  
 * StringUtils.isNoneEmpty(null, "foo")      = false  
 * StringUtils.isNoneEmpty("", "bar")        = false  
 * StringUtils.isNoneEmpty("bob", "")        = false  
 * StringUtils.isNoneEmpty("  bob  ", null)  = false  
 * StringUtils.isNoneEmpty(" ", "bar")       = true  
 * StringUtils.isNoneEmpty("foo", "bar")     = true  
 * </pre>  
 *  
 * @param css  the CharSequences to check, may be null or empty  
 * @return {@code true} if none of the CharSequences are empty or null  
 * @since 3.2  
 */  
public static boolean isNoneEmpty(final CharSequence... css) {  
  
  return !isAnyEmpty(css);  
}    
```

# isBank系列

StringUtils.isBlank()
---------------------

`>>>` `是否为真空值(空格或者空值)`

*   StringUtils.isBlank(null) = true

*   StringUtils.isBlank("") = true

*   StringUtils.isBlank(" ") = true

*   StringUtils.isBlank(“bob”) = false

*   StringUtils.isBlank(" bob ") = false


![](./2024/06/29/isEmpty和isBlank的用法区别居然一半的人答不上来/2.png)
StringUtils.isNotBlank()
------------------------

`>>>` `是否真的不为空`,不是空格或者空值 ,相当于`!isBlank();`

![](./2024/06/29/isEmpty和isBlank的用法区别居然一半的人答不上来/3.png)
StringUtils.isAnyBlank()
------------------------

`>>>`是否包含任何真空值`(包含空格或空值)`

*   StringUtils.isAnyBlank(null) = true

*   StringUtils.isAnyBlank(null, “foo”) = true

*   StringUtils.isAnyBlank(null, null) = true

*   StringUtils.isAnyBlank("", “bar”) = true

*   StringUtils.isAnyBlank(“bob”, “”) = true

*   StringUtils.isAnyBlank(" bob ", null) = true

*   StringUtils.isAnyBlank(" ", “bar”) = true

*   StringUtils.isAnyBlank(“foo”, “bar”) = false


```
 /**  
 * <p>Checks if any one of the CharSequences are blank ("") or null and not whitespace only..</p>  
 * @param css  the CharSequences to check, may be null or empty  
 * @return {@code true} if any of the CharSequences are blank or null or whitespace only  
 * @since 3.2  
 */  
public static boolean isAnyBlank(final CharSequence... css) {  
  
  
  if (ArrayUtils.isEmpty(css)) {  
  
  
    return true;  
  }  
  for (final CharSequence cs : css){  
  
  
    if (isBlank(cs)) {  
  
  
      return true;  
    }  
  }  
  return false;  
}
```

StringUtils.isNoneBlank()
-------------------------

`>>>`是否全部都不包含空值或空格

*   StringUtils.isNoneBlank(null) = false

*   StringUtils.isNoneBlank(null, “foo”) = false

*   StringUtils.isNoneBlank(null, null) = false

*   StringUtils.isNoneBlank("", “bar”) = false

*   StringUtils.isNoneBlank(“bob”, “”) = false

*   StringUtils.isNoneBlank(" bob ", null) = false

*   StringUtils.isNoneBlank(" ", “bar”) = false

*   StringUtils.isNoneBlank(“foo”, “bar”) = true


```
/**  
 * <p>Checks if none of the CharSequences are blank ("") or null and whitespace only..</p>  
 * @param css  the CharSequences to check, may be null or empty  
 * @return {@code true} if none of the CharSequences are blank or null or whitespace only  
 * @since 3.2  
 */  
public static boolean isNoneBlank(final CharSequence... css) {  
  
  
  return !isAnyBlank(css);  
}  
  

