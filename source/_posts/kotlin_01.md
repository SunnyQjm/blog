---
layout:     post                    # 使用的布局（不需要改）
title:      Kotlin use函数的魔法         # 标题
subtitle:                          #副标题
date:       2018-03-30              # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Kotlin
---
> ## 魔法预览

- 实现了Closeable接口的对象可调用use函数
- use函数会自动关闭调用者（无论中间是否出现异常）
- Kotlin的File对象和IO流操作变得行云流水

> ## use函数的原型

```kotlin
/**
 * Executes the given [block] function on this resource and then closes it down correctly whether an exception
 * is thrown or not.
 *
 * @param block a function to process this [Closeable] resource.
 * @return the result of [block] function invoked on this resource.
 */
@InlineOnly
@RequireKotlin("1.2", versionKind = RequireKotlinVersionKind.COMPILER_VERSION, message = "Requires newer compiler version to be inlined correctly.")
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

- 可以看出，use函数内部实现也是通过try-catch-finally块捕捉的方式，所以不用担心会有异常抛出导致程序退出
- close操作在finally里面执行，所以无论是正常结束还是出现异常，都能正确关闭调用者

> ## 来一波对比

- ### 实现读取一个文件内每一行的功能
  - java实现
    ```java
    FileInputStream fis = null;
    DataInputStream dis = null;
    try {
        fis = new FileInputStream("/home/test.txt");
        dis = new DataInputStream(new BufferedInputStream(fis));
        String lines = "";
        while((lines = dis.readLine()) != null){
            System.out.println(lines);
        }
    } catch (IOException e){
        e.printStackTrace();
    } finally {
        try {
            if(dis != null)
                dis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if(fis != null)
                fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```
  - Kotlin实现
    ```java
    File("/home/test.txt").readLines()
            .forEach { println(it) }
    ```
  - 对Kotlin就是可以两行实现。
  - 仔细翻阅readLines这个扩展函数的实现你会发现，它也是间接调用了use，这样就省去了捕捉异常和关闭的烦恼
  - 同样的，经过包装以后你只需要关注读出来的数据本身而不需要care各种异常情况
- ### File的一些其它有用的扩展函数
  ```kotlin
  /**
  * 将文件里的所有数据以字节数组的形式读出
  * Tip：显然这不适用于大文件，文件过大，会导致创建一个超大数组
  */
  public fun File.readBytes(): ByteArray

  /**
  * 与上一个函数类似，不过这个是写（如果文件存在，则覆盖）
  */
  public fun File.writeBytes(array: ByteArray)： Unit

  /**
  * 将array数组中的数据添加到文件里（如果文件存在则在文件尾部添加）
  */
  public fun File.appendBytes(array: ByteArray): Unit


  /**
  * 将文件以指定buffer大小，分块读出（适用于大文件，也是最常用的方法）
  */
  public fun File.forEachBlock(action: (buffer: ByteArray, bytesRead: Int) -> Unit): Unit

  /**
   * Gets the entire content of this file as a String using UTF-8 or specified [charset].
   *
   * This method is not recommended on huge files. It has an internal limitation of 2 GB file size.
   *
   * @param charset character set to use.
   * @return the entire content of this file as a String.
   */
  public fun File.readText(charset: Charset = Charsets.UTF_8): String

  /**
   * Sets the content of this file as [text] encoded using UTF-8 or specified [charset].
   * If this file exists, it becomes overwritten.
   *
   * @param text text to write into file.
   * @param charset character set to use.
   */
  public fun File.writeText(text: String, charset: Charset = Charsets.UTF_8): Unit

  /**
   * Appends [text] to the content of this file using UTF-8 or the specified [charset].
   *
   * @param text text to append to file.
   * @param charset character set to use.
   */
  public fun File.appendText(text: String, charset: Charset = Charsets.UTF_8): Unit

  /**
   * Reads this file line by line using the specified [charset] and calls [action] for each line.
   * Default charset is UTF-8.
   *
   * You may use this function on huge files.
   *
   * @param charset character set to use.
   * @param action function to process file lines.
   */
  public fun File.forEachLine(charset: Charset = Charsets.UTF_8, action: (line: String) -> Unit): Unit


  /**
   * Reads the file content as a list of lines.
   *
   * Do not use this function for huge files.
   *
   * @param charset character set to use. By default uses UTF-8 charset.
   * @return list of file lines.
   */
  public fun File.readLines(charset: Charset = Charsets.UTF_8): List<String>


  /**
   * Calls the [block] callback giving it a sequence of all the lines in this file and closes the reader once
   * the processing is complete.

   * @param charset character set to use. By default uses UTF-8 charset.
   * @return the value returned by [block].
   */
  @RequireKotlin("1.2", versionKind = RequireKotlinVersionKind.COMPILER_VERSION, message = "Requires newer compiler version to be inlined correctly.")
  public inline fun <T> File.useLines(charset: Charset = Charsets.UTF_8, block: (Sequence<String>) -> T): T
  ```
  - 上面的函数都是基于use实现的，可以放心使用，而不用担心异常的发生，并且会自动关闭IO流
