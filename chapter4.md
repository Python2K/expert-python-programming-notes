## <center>第四章  选择好的名字</center>

#### 遵守PEP8风格

1. 常量：使用大写加下划线，这表示这个变量是一个常数值。
2. 命令与使用：常量用来定义程序所依赖的一组值，例如默认的配置文件，好的做法是将所有常量放在包的一个文件内，例子，Django就是放在settings中。
3. 公有与私有变量：对于可变的且可以通过导入自由访问的全局变量，如果他们需要被保护，那么应该使用一个下划线的小写字母，比如：_observers=[],对于函数或类中而言，遵循相同的规则。
4. 函数与方法名：函数与方法名使用小写加下划线。
5. 关于私有属性的争论：对于私有函数与方法，惯例是添加一个前缀下划线。但是如果一个方法有两个前缀下划线，它会在解释器运行时被重命名，以避免与任何子类中的方法产生命令冲突。

```python

class Base(object):
    def __secret(self):
        print("don't tell")
    def pubilc(self):
        self.__secret()

class Derived(Base):
    def __secret(self):
        print('never ever')

print(dir(Derived))
>>>['_Base__secret','_Derived__secret'......]#其中各类中双下划线的函数名被自动修改了，这样可以保证各类中同名的变量名、函数不会冲突
所以当有相同的名称时，首选下双划线
```

最佳做法是编写子类的方法前，查看该类的\_\_mro\_\_解析顺序

6. 特殊方法：以双下划线开始，双下划线结束，对于常规方法而言，永远不要使用这样的命令方式，为了保证可读性，保持在类定义的最开头。
7. 参数：参数名称小写，如果需要加下划线，遵循与变量相同的命令规则。
8. Property:使用小写或小写加下划线。
9. 类：类名始终遵循驼峰式命令法则。
10. 模块和名：除了特殊模块\_\_init\_\_.py之外，模块名都用小写且**不带下划线**。

---

#### 遵循命令指南

1. 用has与is前缀命令布尔元素。
2. 用复数形式命名集合变量。
3. 用显式的名称保存字典。
4. 避免通用的名称。
5. 避免现有的名称：使用上下文中已经存在的名称是不好的做法，对于关键字而言（函数参数），使用后缀下划线是一种避免冲突的方法。

---

#### 参数的最佳实践

函数与方法的签名是代码完整性的保证，它们驱动函数与方法的使用并构建其API，对于参数，同样我们也得小心，通过这三个规则来实现：

* 通过迭代设计构建参数
* 信任参数与测试
* 小心使用魔法参数*args与**kwargs。

**通过迭代设计构建参数**

每个函数都有一个固定的、定义明确的参数列表，那么**鲁棒性**会很好，但是第一个版本中是无法完成的，所以要通过迭代设计来构建，应该能反映出创建该元素所针对的使用场景。以下为示例：

```python
class Service:#版本1
    def _query(self,query,type):
        print('done')
    
    def execute(self,query):
        self._query(query,'EXECUTE')
        
>>>Service().execute('my query')
>>>done#输出done

import logging
class Service:#版本2
    def _query(self, query, type, logger):
        logger('done')
    
    def execute(self,query,logger=logging.info)
    	self._query(query,'EXECUTE',logger)
>>>Service().execute('my query')#旧式调用
>>>Service().execute('my query', logging.warning)
>>>WARNING:root:done
```

如果一个公有元素必须被修改，那么将使用一个deprecation进程，**后面再作介绍**

**信任参数与测试**

考虑到Python动态语言特性，有些开发人员在函数和方**顶部使用断言(assert)**来确保具有正确的内容：这通常是习惯了静态类型，并感觉python缺少了点什么的开发者的做法。！！！这么做有两个主要问题！！！

```python
def division(dividend, divisor):
    assert isinstance(dividend, (int, float))#确保是数字，否则报错
    assert isinstance(divisor, (int, float))
    return dividend/divisor
```

这种方法有两个问题：

* 可读性低
* 每次调用都要进行assert，可能使代码速度变慢

*在任何情况下，断言都必须小心，并且不应该用于使python变成一种静态类型语言。唯一的使用场景应该是保护代码不被无意义的调用*

大多数情况下测试驱动开发（TDD）风格可以提供鲁棒性很好的基础代码-功能测试、单元测试。

别一种方法是模糊测试。

**小心的使用\*args与\*\*kwargs**

这两个魔法方法会破坏函数和方法的鲁棒性，但是当参数列表变的很长的时候，添加魔法方法参数是很吸引人的，但是同时也表示它是一个脆弱的函数与方法，应该被分解或重构。

如果\*args被用于处理元素序列，那么要求传入容器参数(sequence)会更好：

```python
def sum(*args):#可行
    total = 0
    for i in args:
        total += i
    return total

def sum(sequence):#可行
    total = 0
    for i in sequence:
        total += i
    return total
#**kwargs适用同样规则，最好固定参数名，提供有意义的方法签名

def make_sentence(**kwargs):
    noum = kwargs.get('noun', 'Bill')
    return noum

def make_sentence(noum='Bill')
    return noum
```

魔法参数有时无法避免，特别在元编程中，比如装饰器。**在处理函数对于未知数据进行遍历时，魔法参数是十分好用的**。

---

#### 类的名称

类的名称必须简明、精确、易于理解。*常用做法是使用后缀来表示其类型或者特性：*

例如：SQLEngine、MimeTypes、StringWidget、TestCase

对于基类或者抽象类，使用Base与Abstract前缀：BaseCookie、AbstractFormatter

还有一点：尽量不要让类名与属性名称有冗余：比如类名STMP,方法名STMP.stmp_send这样的情况应该避免

---

**模块和包的名称**

模块与包尽量简称，全部小写，且不带下划线：sqilt、postgres

如果实现了一个协议，那么通常会增加lib后缀：import stmplib urllib

在命令空间中保持一致：

```python
from widgets.stringswidgets import TextWidget#不如
from widgets.strings import TextWidget#更好
```

！如果一个模块开始变的复杂，并且包含了许多类，那么好的做法是创建一个包，并将模块的元素划分到其他模块中去,下例：

```python
__init__模块也可以用于将一些API放回顶层，因为它不影响使用，有助于将代码重新组织成更小的部分，例如foo中的__init__模块。
#__init__模块内部
from .module import feature1,feature2
from .module import feature3
这样允许用户直接导入特性：
from foo import feature1, feature2, feature3
但是要注意，这可能会增加循环的依赖性，在__init__模块中的代码将会被实例化。

```

**有用的工具**

Pylint：源代码分析器

Pep8,flake8：风格检查器，也是包装器，添加了一些更有用的特性，例如静态分析和复杂度测量

**Pylint**

此工具除了质量保证方面上，还可以检查源码是否遵循某种命名约定，默认是pep8标准

```python
如果要对pylint进行微调
pylint --generate-rcfile > .pylint#在当前路径下生成pylint文件，这个配置文件自带说明 
```

**pep8,flake8**

pep8只关注风格，如果想要怀一些集成解决方案，pep8更好，需要静态分析，可以使用flake8，它是pep8与其他一些工具的包装器，可以轻松扩展，并提供了更丰富的功能，包括：

* McCabe复杂度测量
* 利用pyflakes做静态分析
* 利用注释禁用整个文件或单行代码



