title=Java异常规范
date=2016-09-29T17:19:32+08:00
type=post
tags=java, exception
status=published
~~~~~~
# 异常规范

## 异常介绍

#### Throwable

所有Exception和Error的父类.

#### Error

致命错误. 项目自身存在问题, 诸如格式有问题, 编译版本不对, 堆栈溢出等, 项目在出现ERROR的情况下是不应该运行的. 同时, 程序遇到Error时, 程序不需要, 通常也是没有能力做处理的, 只能够停止程序针对项目或者运行环境做人工处理才行.
<!-- more -->

#### Exception

区别于Error, 是程序可以自己处理的异常. Exception的子类中, 特殊的RuntimeException被称为**运行时异常**, 也叫**非受检异常**; 其他的子类包括Exception类本身都叫**受检异常**

##### 受检异常

Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

##### 非受检异常(运行时异常)

不需要强制catch或者throw的异常.

## 程序中如何使用异常

程序中我们主要关注**受检异常**和**运行时异常**的使用

**一些原则, 这些原则并不独立, 互相之间有照应或者补充:**

1. 发生可恢复错误的抛出受检异常，程序错误就抛出运行时异常

2. 尽量使用运行时异常. 从保障代码简洁, 清晰, 有意义的角度上来说.

   注意绝对不是无脑把受检异常换为运行时异常. 

   很多时候我们要延迟处理异常: 比如我们的一个受检异常在层次很深的地方抛出, 但是我们在代码层次很高的地方才能做处理, 那么受检异常会出现在代码调用的每一层. 这非常繁琐, 也不清晰.

3. 谨慎抛出受检异常.

   受检异常是不受欢迎的.

   除非你认为你是在强调这个异常, 调用者在大多数情况下需要重点关注这个异常并catch这个异常并做处理. 

   使用运行时异常带来的简洁并不能够弥补开发人员忽略了这个异常带来的问题时.

4. 作为定位是类库的模块, 尽量使用运行时异常, 并对java低层异常封装, 抛出类库特有的概括性的异常. 

   当站在调用者的角度, 可以获悉这个类库有哪几种异常, 出现时代表什么了.  

   移位类库的调用很多时候跟业务没有关系, 当出现错误时, 通常是因为我们的代码漏洞造成的, 这并不能简单通过try_catch进行恢复, 所以尽量不使用受检异常.

5. 作为定位是服务的模块, 可以使用一些受检异常.

   因为当调用服务出现错误, 一般是一个可以解释的业务错误, 如果是想要调用者非常注意的错误, 可以使用受检异常.

   服务的调用一般代码层次比较浅, 并且是和业务比较相关的.

6. 业务异常需要单独封装成新的异常来表达一类或者一个模块的业务错误, 可以使用受检异常. 但也参照1, 2, 3

   可以把一些非业务异常封装成为业务异常, 如果你知道在这个地方这种非业务异常在业务上可以表达一些含义.

   比如某个位置抛出了json解析异常, 我们可以说传入的某个数据格式是错误的.

   为了给大家建立异常体系结构, 业务异常定义为受检异常, 强制让大家关注下.

7. 非业务异常, 代码底层异常, 如果出现的话可以定义为代码bug的, 使用运行时异常

   即使没有catch住的后果是在系统运行时抛给了用户, 也不应该catch. 当然在项目中需要一个最高层次的异常处理, 对非业务异常统一catch记录报警而不要暴露给用户

8. 业务异常如果可以, 不要跨层(跨模块)

   比如

   ```java
   controller -> service -> adaptor -> UC dubbo
   ```

   UC dubbo 抛出的异常, 应该在adaptor或者service做处理封装新的异常, 不要让controller直面UC dubbo的异常.

9. 异常应该携带更多信息. 

   尤其对业务异常来说, 知道异常发生时的业务数据是很重要的, 方便查找定位问题.

10. 在api层(controller层), 将一些业务异常封装为API异常, 这类异常将直接给用户api异常的提示, 且有时可以认为这些异常是正常的, 不需要报警的.

11. 有效的业务异常类划分和异常code定义, 有助于统一处理异常时区别异常的等级合适否需要报警.

   在设计异常时请考虑这一点.

12. 如果不知道自己的异常应该是使用受检异常还是运行时异常, 使用运行时异常.

   先报出错误, 不做对未知的设计 

## 如何处理异常

1. 绝对禁止catch后什么都不做!
2. 在catch之后封装成新异常抛出的时候, 不要记录日志. 因为你抛出了, 会有上层来处理记录日志, 只要没有1这种情况, 总会有信息的. 这里再记录日志就重复了.
3. 在需要时一定要使用上finally
4. 处理异常时记录的日志一般要把异常的堆栈给记录下来.

## 具体实施方案



1. 所有类库

   - [fn-commons](http://git.lianjia.com/fnrd/fn-commons)
   - [common-search](http://git.lianjia.com/fnrd/common-search)
   - [api-common](http://git.lianjia.com/fnrd/api-common)

   每个项目, master打tag, 切新版本分支, 升级大版本, 例如1.0 -> 2.0

   目标: 基本都是用运行时异常,减轻调用负担, 看情况决定是否自定义异常. 类库尽量少记log, 尤其不能记info的log. 这个出log规范的时候再说 

2. 重点项目[fnrd-gte](http://git.lianjia.com/fnrd/fnrd-gte)

   切个exception-refactor分支

   1. 更新类库依赖. 更改由依赖更新引起的代码错误.

   2. 集中在service包:

      - adaptor

        新建一个RPCException, 继承RuntimeException, 替换现有的直接用RuntimeException抛出.

        RPCException可以带一些请求参数信息.

      - api包

        强烈建议使用[api-common](http://git.lianjia.com/fnrd/api-common), 不过涉及较多, 可以逐步改改

      - impl包, 核心流程逻辑

        重点处理的位置, 挨个文件看, 然后从低向上重构

        定义FlowException作为主要流程业务异常, 统一处理时将会使用其中msg通知用户, **非FlowException将会统一封装友好提示**.

        可以定义更加细致的异常继承FlowException异常, 以应对更细致的需要.

      - event, listner

      - 交易, 金融业务相关的增删改查service等.

        可以新定义FnServiceExcetion, TeServiceException, 也作为一类业务异常.

   3. controller

      选择性的对一些异常封装, WebApiException, msg直接给用户显示.