## 第2章　语法最佳实践－类级别以下

#### 语法重要的元素以及使用技巧

* 列表推导式(list comprehension)
* 迭代器(iterator)与生成器(generator)
* 描述符(descriptor)和属性(property)
* 装饰器(decorator)
* with和contextlib

---

#### 字符串与字节

Python3只有一种能够保存文本信息的数据类型，就是**str(string 字符串)**,是不可变序列，保存的是Unicode码位(code point)。其保存的数据类型有明确的限制，就是Unicode文本。

bytes，bytearray只能用字节作为序列值，0<=x<256,绰号前缀有b'foo'代表bytes,看起来与str很像，但是使用list或者tuple转换就能看到其本来面目。

Unicode字符串没有被编码成二进制数据的话是无法保存在磁盘中或通过网络发送的，可以通过str.encode(encoding,errors)方法进行编码，bytes.decode(encoding,errors)方法进行解码。

encoding默认值是'utf-8',errors是指定错误的处理方式，可以取‘strict’(默认值)、'ignore'、'replace'、‘xmlcharref replace’或其他任何进行注册的处理程序(详见codecs模块文档)。

bytes(source,encoding,errors)构造函数，可以创建一个字节序列，如果source是str类型，必须指定encoding。

**str字符串是为文本数据准备的，bytes是为保存与网络传输准备的。**

str,bytes都是不可变序列，微小的改变也需要重新创建一个新的对象实例，**bytearray**是byte的可变版本，可能通过元素赋值进行原地修改，大小也可能动态变化(append、pop、inseer等方法)。

**字符串的拼接操作：使用str += str这种操作，是及其低效的，一般使用join方法(''.join(strings)),当明确知道字符串的数据，可以使用字符串格式化format方法或%运算符。**

---

#### 集合类型

1.列表(list)

2.元组(tuple)

3.字典(dictionary)

4.集合(set)

**Python集合类型不止这4种，标准库扩展了可选列表。**



#### 列表与元组

在当元素位置本身也是信息的数据结构来说，推荐使用元组数据结构，比如坐标(x,y)等

list是对其他对象引用组成的连续数组，不是链表，由于算法复杂度的不同，对于需要真正的链表的场景(双端append和pop复杂度为O(1)的数据结构)，**使用内置collections模块中的deque双端队列**。

![16e3576217fc700abb68a98271fe910ef02dae6b](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/images/list_page1.jpeg)

![16e3576217fc700abb68a98271fe910ef02dae6b-2](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/images/list_page2.jpeg)

**列表操作，当要进行列表操作时候，尽量使用列表推导，更高效简洁**

```python
l = [i for i in range(10) if i %2 ==0]
```

**枚举操作(enumerate),获取列表的索引，使用enumerate函数**

```python
for i,element in enumerate(['one', 'tow']):
    print(i,element)
>>> 0 one
>>> 1 two
>>> 2 three
```

**合并多个列表操作，使用zip函数**

```pytho
s = zip([1,2],[3,4]) #s是一个zip object,是一个迭代器，经过迭代每个元素都是一个tuple
for i in s:
    print(i)
>>> (1,3)
>>> (2,4)
```



**常用语法：序列解包(sequence unpacking),可以使用于任意序列类型**

```pyt
first,second,third = 'first','second','third'
one,*two = 'one','1','2','3'#带星号的表达式会获取序列的剩余部分，是个列表
>>>two
>>>['1','2','3']
one,*two,thred = 1,2,2,3#带星号的表达式会获取中间部分，是个列表.tow的value是[2,2]　
```

#### 字典

字典推导式同样的高效、简短、整洁。

```pytho
>>>{number: None for number in range(6)}
{0: None, 1: None, 2: None, 3: None, 4: None, 5: None}
```



**注意点**：字典的keys()、values()和items()这3个方法在Python3中返回视图对象(view objects)

* keys():返回dict_keys对象，可以查看字典所有的key。
* values():返回dict_values对象，可以查看字典所有的value。
* items():返回dict_items对象，可以查看字典所有的(key,value)二元元组。

*view objects可以动态的查看字典内容，见下例：*

```python
>>>words = {'foo':'bar'}#创建一个字典
>>>items = words.items()＃使用items()方法
>>>words['span'] = 'eggs'＃对原字典添加了key:value
>>>items#查看items，内容动态的跟着变化了
dict_items([('spam','egg'),('foo','bar')])
```

字典操作复杂度：

![dict_page1](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/images/dict_page1.jpeg)

**注意点：**

* 字典的3个基本操作（添加元素，获取元素，删除元素）的最坏情况复杂度的n,n是当前字典元素的个数。
* 复制与遍历字典的操作，其中n是字典**曾经**达到的最大元素数目，而不是当前元素个数，如果对字典修改很大且要频繁遍历某个字典，最好重新创建一个新的字典对象。

字典的元素是无序的，与对象的散列方法、添加顺序都无关。

某些情况下，如果要使用能够保存添加顺序的字典，使用**collections模块下的OrderedDict**,其有一些其他功能，例如使用popitem()方法双端取出元素，使用move_to_end()方法将指定元素移动互某一端，详细功能见官方文档。

#### 集合

* set():是一种可变的、无序的、有限的集合，其**元素**是唯一的，不可变（可哈希的）对象。
* frozenset():是一种不可变的、无序的、可哈希的的集合，其**元素**是唯一的，不可变（可哈希的）对象。

frozenset()因为具有不可变性，所有可以作为字曲的键，在set或者frozenset中不可以再包含set（会引发Type错误），但是可以包含frozenset。

三种创建集合方式:

1. set([1,2,3])
2. {1,2,3}
3. {element for element in range(3)}

**注意**：创建空集合不能使用｛｝，那样是字典，形式上与字典很像，注意区分。

**更多选择**:基本类型是（列表、元组、字典、集合），collections模块提供了更多的选择

* namedtuple():用于创建元组子类的工厂函数，可以通过属性我来访问它的元素索引;
* deque:双端队列，类似列表，是栈和队列的一般化，可以在两端快速添加取出元素;
* ChainMap:类似字典的类，用于创建多个映射的单一视图;
* Counter：字典子类，可对可哈希对象进行计数;
* OrderedDict：字典子类，可以保存元素的添加顺序;
* defaultdict:字典子类，可以通过调用用户自定义的工厂函数来设置缺失值。

---

#### 高级语法

* 迭代器（iterator）
* 生成器（generator)
* 装饰器（decorator）
* 上下文管理器（context　manager）

#### 迭代器

实现了迭代器协议的容器对象的就是迭代器（对象有2个方法\_\__next\_\_与\_\_iter\_\_)，其中__\__next__中返回下一个元素直到raise StopIteration，\_\_iter\_\_返回迭代器自身。

#### 生成器yield

生成器提供了优雅的语法，生成器可以暂停函数并返回一个中间结果，在必要时可以恢复。

生成器是一个特殊的迭代器。

标准库tokenize模块可以从文本流中生成token,并对处理过的每一行都返回一个迭代器，以供后续处理：

```python
import tokenize
>>> reader = open('hello').readline
>>> tokens = tokenize.generate_tokens(reader)
>>>next(tokens)＃可以每次next返回TokenInfo对象
```

**保持代码简单，而不是数据简单，**最好编写多个处理序列值的简单可迭代函数，而不是编写一个复杂函数，同时计算出整个集合的结果。

**生成器的一个重要的特性**:可以利用next函数与调用的代码进行交互，yield变成了一个表达式，而值可以通过send方法来传递：以下是示例

```Python
def ps():
	print('haha')
	while True:
	answer = (yield) #表达式
	if answer is None:
		print('None')
	elif answer.endswith('?'):
		print('>>>???')

p = ps()
next(p)
>>> haha
p.send('what?')#将函数内answer传值为“what？”，变成yield的返回值
>>> >>>???
```

这个函数还可以根据客户端的代码来改变自身的行为，添加了另外2个函数，throw和close

* throw 允许客户端代码发送要抛出的任何类型的异常
* close　作用与throw相同，但会引发特定的异常－－GeneratorExit,在这种情况下，生成器函数必须再次引发GeneratorExit或StopIteration

**生成器是Python中协和、异步并发等其他概念的基础，后面会介绍**



#### 装饰器

1. 作为一个函数，通用模式

```python
def medecorator(function):
	def wrapped(*args,**kwargs):
		#在调用原始函数之前，加工做点什么
		result = function(*args,**kwargs)
		#在函数调用后做点什么
		#并返回结果
		return result
	return wrapped

```

2. 作为一个类，装饰器几乎总是用函数实现，但是某些情况下，使用用户自定义类可能更好。

```python
#非参数化装饰器用作类的通用模式
class DecoratorAsClass:
    def __init_(self,fun):
        self.fun = fun
    def __call__(self,*args,**kwargs):
        #在调用原始函数之前，做点什么
        result = self.fun(*args,**kwargs)
        #在调用函数之后，做点什么，
        #返回结果
        return result 
```

3. 参数化的装饰器

当一个装饰器需要参数，那么对于函数装饰器再包装一层即可，很简单：

```python
def repeat(number=3)#参数默认值为3
	def actual_decorator(func):
        result = None
        def wrapper(*args, **kwargs):
            for _ in range(number):
                result = func(*args,**kwargs):
            return result
        return wrapper
    return actual_decorator

```

**注意，参数化的装饰器即使有默认值，使用时也需要带括号@repeat()**

4. 保存内省的装饰器

使用装饰器常见错误是在使用装饰器时不保存函数的元数据（主要是文档字符串和原始函数名），这样会导致调试装饰过的函数更加困难，也会破坏可能用到的多大多数自动生成文档的工具。

例子说明

```python
def dummy_decorator(func):
    def wrapped(*args,**kwargs):
        '''包装函数内部的文档'''
        return func(*args,**kwargs)
    return wrapped
>>>下面是使用例子
@dummy_decorator
def function_with_important_docstring():
    '''这里是想要保存的重要文档'''

>>>function_with_important_docstring().__name__#显示wrapped函数名
'wrapped'
>>>function_with_important_docstring().__doc__#显示了包装函数的内部文档，并没有保存“想要保存的重要文档”
'包装函数内部的文档'
```

**要解决这个问题，使用functools模块中的wraps()装饰器**

```python
from functools import wraps
#写装饰器
def preserving_decorator(func):
    @wraps(func)
    def wrapped(*args,**kwargs):
        '''包装函数的内部文档'''
        return func(*args,**kwargs)
    return wrapped

@preserving_decorator
def function_with_important_docstring():
    '''这里是我想保存的重要文档'''

>>>function_with_important_docstring().__name__#会正确显示装饰后的函数名元数据
'function_with_important_docstring'
>>>function_with_important_docstring().__doc#会正确显示“想要保存的重要文档”
'这里是我想保存的重要文档'
```

---

**装饰器的用法和有用的例子**
常见的装饰器模式：参数检查、缓存、代理、上下文提供者

**1. 参数检查，可以写一个装饰器，来检查使用的函数的参数是否是指定的类型！！！**

```python
rpc_info = {}
def xmlrpc(in_=(), out=(type(None),)):
    def _xmlrpc(function):
        #注册签名
        func_name = function.__name__
        rpc_info[func_name] = (in_,out)
        def _check_types(elements,types):
            """用来检查类型的子函数"""
            if len(elements) != len(types):
                raise TypeError('argument count is wrong')
            typed = enumerate(zip(elements,types))
            for index,couple in typed:
                arg,of_the_right_type = couple
                if isinstance(arg,of_the_right_type):
                    continue
                raise TypeError('arg #%d should be %s' % (index,of_the_right_type))

        def __xmlrpc(*args):  # 没有允许的关键词
            # 检查输入的内容
            checkable_args = args[1:]
            _check_types(checkable_args, in_)
            # 运行函数
            res = function(*args)
            if not type(res) in (tuple,list):
                checkable_res = (res,)
            else:
                checkable_res = res
            _check_types(checkable_res,out)
                #函数及其类型检查成功
            return res
        return __xmlrpc
    return _xmlrpc

#使用示例

class RPCView:
    @xmlrpc((int, int))
    def meth1(self, int1, int2):
        print('received %d and %d' % (int1, int2))

    @xmlrpc((str,), (int,))
    def meth2(self, phrase):
        print('received %s' % phrase)
        return 12
    
>>># print(rpc_info)
my = RPCView()
my.meth1(1, 2)

# my.meth2('jjj')
```

---

**2. 缓存装饰器**

```python
import hashlib
import pickle
import time

cache = {}#建立一个字典


def memoize(duration=10):#参数装饰器，duration持续时间
    def _memoize(func):
        def is_obsolete(entry, duration):#判断调用时间是否超过了duration
            return time.time() - entry['time'] > duration

        def compute_key(func, args, kw):#创建key,使用pickle来建立hash，冻结所有作为参数传入的对象状态的快捷方式
            key = pickle.dumps((func.__name__, args, kw))
            return hashlib.sha1(key).hexdigest()

        def __memoize(*args, **kw):
            key = compute_key(func, args, kw)
            # 是否已经拥有它了？是否存在于cache内且是否超过duration持续时间
            if key in cache and not is_obsolete(cache[key], duration):
                print('we got a winner')
                return cache[key]['value']
            # 计算
            result = func(*args, **kw)
            # 保存
            cache[key] = {'value': result, 'time': time.time()}
            return result

        return __memoize

    return _memoize


@memoize()
def very_complex_stuff(a, b):
    return a + b


very_complex_stuff(2, 2)

very_complex_stuff(2, 2)
print(cache)
```

**任何情况下，更高效的装饰器会使用基于高级缓存算法的专用缓存库**

---

**3. 代理**

代理装饰器使用全局机制为标记和注册函数，

```python
class User(object):#定义一个User类，内含规则
    def __init__(self, roles):
        self.roles = roles

class Unauthorized(Exception):#自定义一个异常类
    pass

def protect(role):#role规则参数
    def _protect(function):
        def __protect(*args, **kwargs):
            user = globals().get('user')#获取全局变量user
            if user is None or role not in user.roles:#如果user为空或者不在规则参数中，返回自定义异常
                raise Unauthorized('I wont tell you')
            return function(*args, **kwargs)
        return __protect
    return _protect
```

这一模型常用于python web框架用于定义发布类的安全性

下面是使用示例，当一个用户被保存在一个全局变量中，在方法被访问时装饰器会检查他的角色

```python
tarek = User(('admin', 'user'))
bill = User(('user'))
class MySecret(object):
    @protect('admin')
    def waffle_recipe(self):
        print('use tons of butter')
        
>>>these_are = MySecrets()
>>>user = tarek#全局变量用户user
>>>these_are.waffle_recipe()
use tons of butter

>>>user = bill
>>>these_are.waffle_recipe()
Traceback(most recent call last)
File "<stdin>", line 1, in <module>
File "<stdio>", line 7, in wrap
__main__.Unauthorized: I wont tell you
```

---

**4. 上下文提供者**

上下文装饰器确保函数可以运行在正确的上下文中，或者函数前后运行一些代码，它设定并复位一个特定的执行环境。举例，当一个数据项需要在多个线程之间共享时，需要一个锁来保护它避免多次访问，这个锁可以在装饰器中编写，如下：

```python
def synchronized(func):
    def _synchronized(*args, **kwargs):
        lock.acquire()#获取锁
        try:
            return func(*args, **kwargs)
        finally:
            lock.release()#释放锁
    return _synchronized

@synchronized
def thread_safe():#确保锁定资源
    pass
```

上下文装饰器经常会被上下文管理器（with语句）替代。

---

**上下文管理器—with语句**

**try…finally**为了即使在出现错误的情况下，也可以执行**某些清理代码**,try…finally很有用，一般使用场景如下：

* 关闭一个文件
* 释放一个锁
* 创建一个临时的代码补丁
* 在特殊环境中运行受保护的代码

with语句一般语法的最简单的形式：

```python
with context_manager:
    #代码块
...

with context_manager as context:#使用as子句保存为局部变量
    #代码块
...
    
with A() as a,B() as b:#同时使用多个上下文管理器
...
```

**ps：下面这部分实际使用场景还不太了解，作个记录**

1. **作为一个类**，任何实现了context manager protocol 的对象都可以用作上下文管理器，该协议包涵了2个特殊方法：\_\_enter\_\_(self),\_\_exit\_\_(self,exc_type,exc_value,traceback)

   with语句的执行过程如下：

   > 调用enter方法，任何返回值都会绑定到指定的as子句
   >
   > 执行内部代码块
   >
   > 调用exit方法

   exit方法中3个参数：如果没有出现错误，3个参数都被设定为None，出错时，exit不应该重新引发这个错误，详细内容另查。。。。。。

   **以下为一个简单例子，可以说明其工作原理**

   ```python
   class ContextIllustration:
       def __enter__(self):#先执行enter方法，如果有返回值，都会绑定到as子句
           print('entering context')
       def __exit__(self,exc_type,exc_value,trace_back):
           print('leaving context')
           if exc_type is None:#如果没有错误，参数会设定为None
               print('no error')
           else:
               print("with an error {}".format(exc_value))
              
   举例个错误情况
   >>>with ContextIllustration():
           raise RuntimeError("raised with in 'with")#"raised with in 'with"就是exc_value的值
   以上会引发异常运行__exit__中的else下面的代码
   ```

   2. 作为一个函数—contextlib模块

   标准库中有contextlib模块，提供了与上下文管事器一起使用的辅助函数，它最有用的是contextmanager装饰器，可以在一个函数中提供\_\_enter\_\_和\_\_exit\_\_两部分，蹭用yield分开，将ContextIllustration类改写如下：

   ```python
   from contextlib import contextmanager
   @contextmanager
   def context_illustration():
       print('entering context')
       try:
           yield
       except Exception as e:#如果有异常
           print('leaving context')
           print("with an error {}".format(exc_value))
   		#需要再次抛出异常
           raise 
       else:#如果没有异常
           print('leaving context')
           print('no error')
   
   ```

   ---

   **一些其他语法元素**

   1. for …else语句，解释，当for循环自然结束时，运行else子句，如果在for循环中有break，则不执行else子句。某些情况，当执行完for循环后，需要删除一些“哨兵(sentinel)变量”，如果出现break时用户想要保存信息，可能会需要这些变量，就用的上这种语句了。
   2. 函数注解：注解并没有被广泛使用

   ```python
   def f(ham:str, eggs:str = 'eggs') -> str:
       pass
   
   >>>print(f.__annotations__)#注解可以通过__annotations__属性获取，是个字典，在应用运行期间可以获取
   {'return': <class 'str'>, 'eggs':<class 'str'>, 'ham':<class 'str'>}
   
   ```

   







