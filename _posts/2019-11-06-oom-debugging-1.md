---
layout: post
title:  OOM的排查过程
categories: [Java]
tags: [OOM,Java]
comments: true
---

好久不写博客了，现在抽时间记录一下一次OOM异常的排查过程

<!--more-->

##### OOM发生场景：

在做Poi导出excel文件的时候，连续执行多次（2次以上）之后就出现OOM；JVM参视配置堆内存设置为500M。

导出文件大小为30M左右，JDK1.8。

##### 排查过程：

首先需要获取内存溢出时的内存快照，然后通过eclipse的MemoryAnalyzer性能分析插件来分析堆内存的信息。

###### -- 获取内存快照（dump文件）

配置jvm参数

```xml
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof
-XX:+PrintGCDetails
```

###### --分析dump文件

通过eclipse的MemoryAnalyzer软件加载dump文件

![mybatis-1](/img/posts/oom-01-01.png)

###### --自我推测（Java代码）

```java
exportExcelTaskDao.insertTaskInfo(exportExcelForm.getExportTaskModel());
List<Map<String, Object>> data = getData(exportId, condition);
exportExcelForm.setData(data);
String fileUrl = PoiUtil.uploadFile(exportExcelForm);
exportExcelTaskDao.updateTaskStatus(1, fileUrl, exportExcelForm.getExportTaskModel().getId());
```

上面是导出文件的核心代码片段

其中*List<Map<String, Object>> data = getData(exportId, condition);*是通过mybatis查出数据库的数据结果是自动映射到Map里面。

通过上面分析dump文件可以看出导致内存不足的罪魁祸首应该是这个对象。

###### --分析结论

**最后保存到文件中只有30M左右的数据怎么却占用了约350M的内存呢？**

为什么会出现这种情况呢，大家都知道Map是由扩容机制的。1.8中HashMap的源码中由如下参数

- HashMap的扩容机制分析

```java
/**
 * The load factor used when none specified in constructor.
 */
 static final float DEFAULT_LOAD_FACTOR = 0.75f;
 
 /**
  * The next size value at which to resize (capacity * load factor).
  *
  * @serial
  */
  // (The javadoc description is true upon serialization.
  // Additionally, if the table array has not been allocated, this
  // field holds the initial array capacity, or zero signifying
  // DEFAULT_INITIAL_CAPACITY.)
  int threshold;
```

| 名称            | 用途                                                         |
| :-------------- | :----------------------------------------------------------- |
| initialCapacity | HashMap 初始容量                                             |
| loadFactor      | 负载因子                                                     |
| threshold       | 当前 HashMap 所能容纳键值对数量的最大值，超过这个值，则需扩容 |

默认情况下，HashMap的初始容量是16，加载因子是0.75。达到容量的0.75被之后就会2倍扩容。

而导出的excel文件中的数据为200847条，16列左右的数据。

所以必然完成至少一次的扩容。而在堆中创建一个对象，分为三步，申请内存空间、获取地址号、初始化数据。虽然最后每个Map中真实的存储16个key-value但是，在堆中占用的内存已然固定。即是按着最后一次扩容的大小算的。

再加上20万条数据，所以再加上。ArrayList的扩容机制。

- ArrayList的扩容机制分析

```java
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

```

上面是ArrayList 的部门源码。可以看出初始容量为10，在进行add添加元素时，当超过现有容量时，会进行2倍扩容（这里使用了位运算提高效率）。

- 总结

综上，可以想象到，其实是有可能出现几十兆的excel文件加载到内存中映射到map中却是几百兆。



----

需要注意的是：**上面的结论只是合理推测，后续会对此进行验证（将Map以自定义实体类替代）**。不过当时统计的数据为了方便，mybatis的持久层都是使用List<Map<String,Object>>  这种类型接收结果集的，其实在一些公司的规范（如阿里巴巴的开发手册）中是声明禁止使用Map来接收对象的，这便是其中一个原因。所以不要嫌麻烦，最好还是准确的建个实体类。





