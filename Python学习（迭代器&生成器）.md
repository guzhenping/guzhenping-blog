# yield/生成器/协程

## 前言
一直对Python的生成器、yield相关的东西比较晕。也终于到了这一天，算算总账，一口气理解这些东西。

这篇文章的背景是对yield/生成器/协程知识的总结，用于自我提升。请静下心来读，走马观花就不会有自己的思考。

懂分享的人，一定会快乐。笔者的内心期待着大家一起进步。所以，一起来分享自己的理解，留下你的评论吧。

## 行文介绍
因为这是一整块知识，请不要拆散理解（好处是更易融会贯通）。所以，从头到尾的必备知识：

- 可迭代对象（iterable）
- 迭代器（iterator）
- 迭代（iterator）
- yield表达式
- 生成器（generators）
  
本文会分别介绍上面的知识，最后介绍协程。剧透一下，协程和生成器很相似，所以学习协程，必备生成器相关的知识。

## 啥是iterable？
Python中任意的对象，只要它定义了可以返回一个迭代器的__iter__方法，或者定义了可以支持下标索引的__getitem__方法，那么它就是一个可迭代对象。

简单说，可迭代对象就是能提供迭代器的任意对象。

## 啥是iterator？
任意对象，只要定义了\_\_iter\_\_方法和next(Python2) 或者__next__（Python3）方法，它就是一个迭代器。\_\_iter\_\_()返回迭代器对象本身；next()或者\_\_next\_\_()返回容器的下一个元素，在结尾时引发StopIteration异常退出。

对于可迭代对象，可以使用内建函数iter()来获取它的迭代器对象。

```
# python3中
>>> test = [1,2,3] 
>>> item = iter(test)    # 获取迭代器对象
>>> print(item)    # 打印该对象的类型、地址
<list_iterator object at 0x10231ab38>
>>> item.__next__()    # 获取第一个元素
1
>>> item.__next__()    # 获取第二个元素
2
>>> item.__next__()    # 获取第三个元素
3
>>> item.__next__()    # 获取StopIteration
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

可以对string和dict做上面的事情，看看效果。扩展一点：在for循环中，for语法会自动调用迭代器的__next__()或next()，并能在遇到StopIteration时正常退出循环。

作为强化，给出一个自定义iterator的实例：

```
class MyIteratorFab():
    """Python3实现生成斐波那契数Fibonacci
       输出最后一个数据不大于max
    """
    def __init__(self, max):
        self.max = max
        self.a = 0
        self.b = 1

    def __iter__(self):
        return self

    def __next__(self):
        """ 返回容器中下一个元素,没有(符合标准的)元素后, 抛出StopIteration
            python2中请将__next__替换成next
        """
        if self.b <= self.max:
            r = self.b    # 用于记录倒数第二个b的值,该值才是小于self.max的
            self.a, self.b = self.b, self.a + self.b
            return r    # 不可返回self.b, 该值会比self.max大一点点
        else:
            raise StopIteration()    # 这条代码很重要,删除这不能正常退出for循环

# 测试
test = MyIteratorFab(100)
for item in test:
    print(item)
    
#############################################
# 附一段生成fibonacci数的函数，同上述自定义迭代器是
# 同一功能.
#############################################
def fib_func2(max):
    """一个普通的的生产fibonacci数的函数
       输出最后一个数据不大于max
    """
    a, b = 0, 1
    while b <= max:
        print(b)
        a, b = b, a+b
```

最后，在[《Python迭代器和生成器》](http://python.jobbole.com/81881/)一文中提到过：对于一个可迭代对象，如果它本身又是一个迭代器对象，那么没办法支持多次迭代。感兴趣可戳过去阅读。

文中给出该问题的解法是，对迭代器对象类再包一个可迭代对象，实现多次迭代。拿上述MyIteratorFab()自定义迭代器举例：

```
# 问题：
fab = MyIteratorFab(10)
print([item for item in fab])    # output: [1, 1, 2, 3, 5, 8]
print([item for item in fab])    # output: []

# 解决方法：
class BetterFab():
    """实现生成斐波那契数Fibonacci
       输出最后一个数据不大于max
       可多次迭代
    """
    def __init__(self, max):
        self.max = max

    def __iter__(self):
        return MyIteratorFab(self.max)
        
# 测试
fab2 = BetterFab(10)
print([item for item in fab2])    # output: [1, 1, 2, 3, 5, 8]
print([item for item in fab2])    # output: [1, 1, 2, 3, 5, 8]

# 暴力的解法，但不推荐：
print([item for item in MyIteratorFab(10)])    # output: [1, 1, 2, 3, 5, 8]
print([item for item in MyIteratorFab(10)])    # output: [1, 1, 2, 3, 5, 8]
```
到此，iterator就算是说完了。这是三个以itera*开头的概念中最核心的一个概念。当然，它也是生成器（generator）的基础。

## 啥是iteration？
提醒一下，这是一个名词。用简单的话讲，它就是从某个地方（比如一个列表）取出一个元素的过程。当我们使用一个循环来遍历某个东西时，这个过程本身就叫迭代。


## 如何理解yield表达式？
下面的内容摘自Python3官方文档翻译：
> ```
> yield_atom ::=  "(" yield_expression ")"
yield_expression ::=  "yield" [expression_list | "from" expression]
> ```
> yield表达式仅在定义生成器函数时使用，因此只能用在函数定义的主体中。在函数体中使用yield表达式会使该函数成为生成器。

>当生成器函数被调用时，它返回一个称为生成器的迭代器。然后，生成器控制生成器函数的执行。当生成器的一个方法被调用时，执行开始。此时，执行进行到第一个yield表达式，在那里它被再次挂起，将expression_list的值返回给生成器的调用者。挂起，我们的意思是保留所有局部状态，包括局部变量的当前绑定，指令指针，内部计算栈和任何异常处理的状态。当通过调用其中一个生成器的方法来恢复执行时，函数可以像yield表达式只是另一个外部调用一样继续进行。恢复后的yield表达式的值取决于恢复执行的方法。如果使用\_\_next\_\_()（通常通过for或next()内置函数），则结果为None。否则，如果使用send()，则结果将是传递到该方法的值。
>
>摘自：http://python.usyiyi.cn/translate/python_352/reference/expressions.html#yieldexpr

yield 是一个类似 return 的关键字，只是这个函数返回的是个生成器。

另外，官网也提到yield其实和Coroutine类似：
>所有这些使生成器函数与协程非常相似；它们产生多次，它们具有多个入口点并且它们的执行可以被挂起。唯一的区别是生成器函数不能控制在它yield后继续执行的位置；控制总是转移到生成器的调用者。

## 啥是generators？
下面的内容摘自Python3官方文档翻译：
> ```
> generator_expression ::=  "(" expression comp_for ")"
> ```
> 生成器表达式产生一个新的生成器对象。它的语法与推导式的语法相同，除了它被括在括号中而不是括号或花括号中。
> 
>当生成器对象调用\_\_next\_\_()方法时，生成器表达式中使用的变量将被懒惰地计算（以与正常生成器相同的方式）。但是，最左边的for子句会立即被求值，所以它产生的错误可以在生成器表达式代码中的任何其它可能的错误之前发现。后续for子句无法立即计算，因为它们可能取决于之前的for循环。例如：(x*y for x in range(10) for y in bar(x))。
>
>摘自：http://python.usyiyi.cn/translate/python_352/reference/expressions.html#yieldexpr

生成器也是一种迭代器，但是你只能对其迭代一次。这是因为它们并没有把所有的值存在内存中，而是在运行时生成值。你通过遍历来使用它们，要么用一个“for”循环，要么将它们传递给任意可以进行迭代的函数和结构。大多数时候生成器是以函数来实现的。然而，它们并不返回一个值，而是yield(暂且译作“生出”)一个值。

生成器最佳应用场景是：你不想同一时间将所有计算出来的大量结果集分配到内存当中，特别是结果集里还包含循环。


## 协程（Coroutine）
常见于合作式多任务，迭代器，无限列表，管道。

协程是基于单个线程的，Python的线程是基于单核的并发实现。

> 并发对应计算机中充分利用单核（一个CPU）实现（看起来）多个任务同时执行。实现并发编程可以用多进程、多线程、异步、协程。

从上面这段引用可以看出：为啥很多人讨论协程与其他并发编程方式的异同优劣？但是今天我们不聊并发，所以关注该话题的童鞋请自己Google文章。

深挖Coroutine的本质:
>allowing multiple entry points for suspending and resuming execution at certain locations.

允许多个入口对程序进行挂起、继续执行等操作。

这和生成器很相似，但有区别：

- 生成器是数据的生产者
- 协程则是数据的消费者

协程会消费掉发送给它的值。

协程常用的方法：
next()
send()
close()

## 结语
有件事情必须要说明白：

我是宇宙中微不足道的一粒沙。即使作为沙粒，我想浸在湛蓝的海水中，也想沐浴在灿烂的阳光下，更想陪着孩童们构建沙滩城堡。自然清楚自己是一粒沙，但还是想要这样的生活。

所以，我愿意写这些微不足道的东西，来分享自己的价值。尽管，我也微不足道。


## 参考资料

- [Python 线程与协程](http://blog.rainy.im/2016/04/07/python-thread-and-coroutine/)
- [Python关键字yield的解释](http://pyzh.readthedocs.io/en/latest/the-python-yield-keyword-explained.html)
- [Python yield 使用浅析](http://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/)
- [Python高级编程之生成器(Generator)与coroutine(一):Generator](http://www.cnblogs.com/rio2607/p/4440122.html)
- [Python高级编程之生成器(Generator)与coroutine(二):coroutine介绍](http://www.cnblogs.com/rio2607/p/4456332.html)
- [Python高级编程之生成器(Generator)与coroutine(三):coroutine与pipeline(管道)和Dataflow(数据流_](http://www.cnblogs.com/rio2607/p/4472456.html)
- [Python高级编程之生成器(Generator)与coroutine(四):一个简单的多任务系统](http://www.cnblogs.com/rio2607/p/4570353.html)
- [Python迭代器和生成器](http://python.jobbole.com/81881/)