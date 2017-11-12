> JDK8 发布很久了，它提供了许多吸引人的新特性，能够提高编程效率。
>
> 如果是新的项目，使用 JDK8 当然是最好的选择。但是，对于一些老的项目，升级到 JDK8 则存在一些兼容性问题，是否升级需要酌情考虑。
>
> 近期，我在工作中遇到一个任务，将部门所有项目的 JDK 版本升级到 1.8 （老版本大多是 1.6）。在这个过程中，遇到一些问题点，并结合在网上看到的坑，在这里总结一下。

## FAQ

### sun.\* 包缺失问题

JDK8 不再提供 `sun.*` 包供开发者使用，因为这些接口不是公共接口，不能保证在所有 Java 兼容的平台上工作。

使用了这些 API 的程序如果要升级到 JDK 1.8 需要寻求替代方案。

虽然，也可以自己导入包含 `sun.*` 接口 jar 包到 classpath 目录，但这不是一个好的做法。

需要详细了解为什么不要使用 `sun.*` ，可以参考官方文档：[Why Developers Should Not Write Programs That Call 'sun' Packages](http://www.oracle.com/technetwork/java/faq-sun-packages-142232.html)

### 默认安全策略修改

升级后估计有些小伙伴在使用不安全算法时可能会发生错误，so，支持不安全算法还是有必要的

找到$JAVA_HOME下 `jre/lib/security/java.security` ，将禁用的算法设置为空：`jdk.certpath.disabledAlgorithms=` 。

### 第三方jar包无法使用

有些第三方 jar 包基于非 JDK8 版本编译，可能会存在兼容性问题。

这种情况只能具体问题具体分析，下面列举几个常用 jar 包。

- 查找组件用到了 mvel，mvel 为了提高效率进行了字节码优化，正好碰上 JDK8 死穴，所以需要升级。

```xml
<dependency>
  <groupId>org.mvel</groupId>
  <artifactId>mvel2</artifactId>
  <version>2.2.7.Final</version>
</dependency>
```

- javassist

```xml
<dependency>
  <groupId>org.javassist</groupId>
  <artifactId>javassist</artifactId>
  <version>3.18.1-GA</version>
</dependency>
```

> **注意**
>
> 有些部署工具不会删除旧版本 jar 包，所以可以尝试手动删除老版本 jar 包。

### JVM参数调整

在jdk8中，PermSize相关的参数已经不被使用：

```
-XX:MaxPermSize=size

Sets the maximum permanent generation space size (in bytes). This option was deprecated in JDK 8, and superseded by the -XX:MaxMetaspaceSize option.

-XX:PermSize=size

Sets the space (in bytes) allocated to the permanent generation that triggers a garbage collection if it is exceeded. This option was deprecated un JDK 8, and superseded by the -XX:MetaspaceSize option.
```

JDK8 中再也没有 `PermGen` 了。其中的某些部分，如被 intern 的字符串，在 JDK7 中已经移到了普通堆里。**其余结构在 JDK8 中会被移到称作“Metaspace”的本机内存区中，该区域在默认情况下会自动生长，也会被垃圾回收。它有两个标记：MetaspaceSize 和 MaxMetaspaceSize。**

-XX:MetaspaceSize=size

> Sets the size of the allocated class metadata space that will trigger a garbage collection the first time it is exceeded. This threshold for a garbage collection is increased or decreased depending on the amount of metadata used. The default size depends on the platform.

-XX:MaxMetaspaceSize=size

>  Sets the maximum amount of native memory that can be allocated for class metadata. By default, the size is not limited.  The amount of metadata for an application depends on the application itself, other running applications, and the amount of memory available on the system.

以下示例显示如何将类类元数据的上限设置为 256 MB：

XX:MaxMetaspaceSize=256m

## 资料

- [Compatibility Guide for JDK 8](http://www.oracle.com/technetwork/java/javase/8-compatibility-guide-2156366.html)
- [Compatibility Guide for JDK 8 中文翻译](https://yq.aliyun.com/articles/236)
- [Why Developers Should Not Write Programs That Call 'sun' Packages](http://www.oracle.com/technetwork/java/faq-sun-packages-142232.html)