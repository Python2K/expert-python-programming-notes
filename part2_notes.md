##第2章　语法最佳实践－类级别以下

####语法重要的元素以及使用技巧

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



####列表与元组

在当元素位置本身也是信息的数据结构来说，推荐使用元组数据结构，比如坐标(x,y)等

list是对其他对象引用组成的连续数组，不是链表，由于算法复杂度的不同，对于需要真正的链表的场景(双端append和pop复杂度为O(1)的数据结构)，**使用内置collections模块中的deque双端队列**。

![16e3576217fc700abb68a98271fe910ef02dae6b](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/list_page1.jpeg)

![16e3576217fc700abb68a98271fe910ef02dae6b-2](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/list_page2.jpeg)

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

![dict_page1](https://raw.githubusercontent.com/Python2K/expert-python-programming-notes/master/dict_page1.jpeg)

**注意点：**

* 字典的3个基本操作（添加元素，获取元素，删除元素）的最坏情况复杂度的n,n是当前字典元素的个数。
* 复制与遍历字典的操作，其中n是字典**曾经**达到的最大元素数目，而不是当前元素个数，如果对字典修改很大且要频繁遍历某个字典，最好重新创建一个新的字典对象。

字典的元素是无序的，与对象的散列方法、添加顺序都无关。

某些情况下，如果要使用能够保存添加顺序的字典，使用**collections模块下的OrderedDict**,其有一些其他功能，例如使用popitem()方法双端取出元素，使用move_to_end()方法将指定元素移动互某一端，详细功能见官方文档。

