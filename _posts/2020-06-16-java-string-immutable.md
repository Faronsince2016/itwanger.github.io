---
layout: post
category: java
title: 再见，Java字符串是不可变的
tagline: by 沉默王二
tags: 
  - java
---

最近，又有好几个小伙伴问我这个问题：“二哥，为什么 Java 的 String 要设计成不可变的啊？”说实话，这也是一道非常经典的面试题，面试官超喜欢问。我之前写过这方面的文章，现在读起来似乎不太满意，所以我决定再啰嗦最后一次，交出一份更满意的答卷，让小伙伴们在面试官面前更从容一些，更有底气一些。

<!--more-->

![](http://www.itwanger.com/assets/images/2020/06/java-string-immutable-01.gif)

关于不可变对象，还有这样一个小故事。Java 之父詹姆斯高司令曾在一次采访中被问及这样一个问题：“高司令，应该什么时候使用不可变对象啊？”你猜高司令怎么回答？

>如有可能，我愿意任何时候都使用不可变对象。

这就是高司令的答案，那有的小伙伴可能不服，老人家会说中文，你瞎扯吧你。也对哈，那就上英文呗：

>I would use an immutable whenever I can.

这下彻底被打服了吧？老人家还说，不可变有着非常强大的功能，比如说，缓存、安全性、高性能等等。

### 01、什么是不可变对象

不可变对象在创建后，它的内部状态会保持不变，这就意味着，一旦我们将一个对象分配给一个变量，就无法再通过任何方式更改对象的状态了。

关于不可变对象的更多信息，可以查看我之前写的另外一篇文章——[这次要说不明白immutable类，我就怎么地](https://mp.weixin.qq.com/s/wbdV9rV60AwWiiTEBYPP7g)，看完啥都明白了。你看，写系列文章的好处就是这样，不需要重复造轮子，用到的时候直接搬出来套上就行了。

### 02、为什么 String 是不可变的

重点来了啊，为什么 String 是不可变的？原因可以从四个方面说起，缓存、安全性、同步和高性能。

**1）字符串常量池**

字符串恐怕是 Java 中最常用的数据形式了，如果字符串非要谦虚地说自己是老二，就没有人敢说自己是老大。

因此，把字符串缓存起来，并且重复使用它们会节省大量堆空间（堆内存用来存储 Java 中的对象，无论是成员变量、局部变量，还是类变量，它们指向的对象都存储在堆内存中），因为不同的字符串变量引用的是字符串常量池中的同一个对象。这也正是字符串常量池存在的目的。

字符串常量池是 Java 虚拟机用来存储字符串的一个特殊的区域，由于字符串是不可变的，因此 Java 虚拟机可以在字符串常量池中只为同一个字符串存储一个字符串副本来节省空间。

字符串常量池的主要使用方法有两种：

- 直接使用双引号声明出来的字符串对象会直接存储在常量池中。
- 否则，可以使用 String 类提供的 `intern()` 方法强制将当前字符串放入常量池中——常量池中查询不到当前字符串。

来看下面这段代码：

```java
String s1 = "沉默王二";
String s2 = "沉默王二";

System.out.println(s1 == s2); // true
```

由于字符串常量池的存在，所以两个不同的变量都指向了池中同一个字符串对象，从而节省了稀缺的内存资源。如果是通过 new 关键字创建的对象，则需要新的堆空间。

![](http://www.itwanger.com/assets/images/2020/06/java-string-immutable-02.png)

放心，关于字符串常量池，后面有时间的话，我再单独写一篇文章详细地说一说。

**2）安全性**

字符串在 Java 应用程序中的使用范围非常广，几乎无处不在，比如说存储用户名、密码、数据库连接地址等等这些非常敏感的信息，因此，必须要保证 String 类的绝对安全性。

来考虑一下下面这段代码：

```java
void criticalMethod(String userName) {
    // 检查用户名是否合法
    if (!isAlphaNumeric(userName)) {
        throw new SecurityException(); 
    }
     
    // 初始化数据库连接
    initializeDatabase();
     
    // 准备修改用户状态
    connection.executeUpdate("UPDATE members SET status = 'active' " +
      " WHERE username = '" + userName + "'");
}
```

通常情况下，用户名由客户端传递到服务器端，服务器端接收后要先对用户名进行检查，再进行其他操作，因为客户端传递过来的信息不一定值得信任。

如果字符串是可变的，那么我们在执行 `executeUpdate` 更新数据库的时候，就有点不放心，因为即便是安全性检查通过了，字符串仍然有可能被修改。

在调用 `isAlphaNumeric()` 方法进行安全性检查期间，userName 的值仍然有可能被 `criticalMethod()` 方法的调用者进行篡改，就容易造成 SQL 注入。

但如果字符串是不可变的，这方面的担忧就不存在了。因为在执行更新之前，字符串的值是确定的，就是我们检查安全性之后的值。

**3）线程安全**

由于字符串是不可变的，因此可以在多线程之间共享，如果一个线程把字符串的值修改为另外一个，那么就会在字符串常量池中创建另外一个字符串，原有的字符串仍然会保持不变。

不过，很遗憾，我还不知道怎么从代码层面上去证明这一点，只能纯理论 yy 一下。小伙伴谁有办法的，教教我，在线等的那种。

**4）哈希码**

字符串广泛应用于 HashMap、HashTable、HashSet 等需要哈希码作为键的数据结构中，在对这些哈希表进行操作的时候，需要频繁调用 `hashCode()` 方法来获取键的哈希码。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

由于字符串是不可变性，这就保证了键值的哈希值不会发生改变，因此在第一次调用 String 类的 `hashCode()` 方法时，就对哈希值进行了缓存，此后，就一直返回相同的值。

```java
/** Cache the hash code for the string */
private int hash; // Default to 0

public int hashCode() {
    int h = hash;
    if (h == 0 && !hashIsZero) {
        h = isLatin1() ? StringLatin1.hashCode(value)
                : StringUTF16.hashCode(value);
        if (h == 0) {
            hashIsZero = true;
        } else {
            hash = h;
        }
    }
    return h;
}
```

由于哈希值被缓存了，这在另外一种层面上提高了哈希表的访问性能，因为哈希值不用重新计算了。

假如字符串是可变的，那就意味着哈希码会有多个，在通过键获取值的时候，就不一定能够获取到对的值了。

你看，字符串常量池的存在，哈希码的存在，在很大程度上提高了程序的性能。

### 03、总结

好了，我亲爱的小伙伴们，以上就是本文的全部内容了。我相信你一定对字符串的不可变性有了充足的了解，由于字符串是不可变的，因此我们可以将它看作是一个特殊的基本数据类型，哪怕是在多线程的环境下，也不用担心它的值是否会发生改变。


如果觉得文章对你有点帮助，请微信搜索「 **沉默王二** 」第一时间阅读。

>本文已收录 GitHub，[**传送门~**](https://github.com/qinggee/itwanger.github.io) ，里面更有大厂面试完整考点，欢迎 Star。

我是沉默王二，一枚有颜值却靠才华苟且的程序员。**关注即可提升学习效率，别忘了三连啊，点赞、收藏、留言，我不挑，嘻嘻**。