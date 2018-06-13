## <center>第三章 语法最佳实践—类级别以上</center>

#### 本章主题

* 子类化内置类型
* 访问超类中的方法
* 使用property和slot
* 元编程

---

#### 子类化内置类型

所有的类，都有一个共同的祖先，就是object的**内置类型(也就是没有显示的指定父类，但是是自定义类的祖先)**。

一个不允许添加相同值的类：

```python

class DistinctError(ValueError):
    '''如果向distincdict添加重复值，会引发这个错误'''
    pass


class Distincdict(dict):
    '''不接受重复值的字典'''
    def __setitem__(self, key, value):
        if value in self.values():#如果这个值已经在字典中了
            if (key in self and self[key] != value) or key not in self:#只接受已经存在的键值对覆盖添加
                raise DistinctError("this value already exists for different key")
        super(Distincdict,self).__setitem__(key,value)
        #也可以简化写成super().__setitem__(key,value)
```

内置类型覆盖了大部分的使用场景，还有collections模块中还有很多有用的容器。

---

#### 访问超类中的方法

***super***是一个**内置类**，用于访问属于某个对象的超类的属性，看上去似乎是个函数，但是其他是一个内置类。

在定义类时，方法内部使用super可以简化super(),括号中可以不传任何参数，但是如果不在方法内部使用，必须给出参数，如下：

```python
anita = Sister()
>>>super(anita.__class__, anita).says()
```

***!!!***~~super有个很重要的一点注意，其第二个参数是可选的，如果只提供了一个参数，那么将返回一个unbound类型，这一点在与classmethod一起使用时**特别有用**，以下为示例：~~

```python
class Pizza:
    def __init__(self,toppings):
        self.toppings = toppings

    def __repr__(self):
        return "Pizza with" + "and".join(self.toppings)

    @classmethod
    def recommend(cls):
        """推荐任何馅料(toppings)的某种Pizza"""
        return cls(['spam', 'ham', 'eggs'])#午餐肉，火腿，鸡蛋
    
class VikingPizza(Pizza):
    @classmethod
    def recommend(cls):
        """推荐与super相同的内容，但是多加了spam"""
        recommended = super(VikingPizza).recommend()
        recommended.toppings +=['spam'] * 5
        return recommended
    !!!!!!以上代码测试不通过，super缺少第二个参数无法调用，查询中。。。。。。
```

~~**注意，零参数的super()也可以用于被@classmethod装饰器装饰的方法，无参数调用的super()被看作仅仅定义了第一个参数。**~~划线部分未通过测试，暂时记录。。。。。。

如果面对多重继承，super将变的难以使用，理解何时应避免使用super以及**方法解析顺序(method Resolution Order, MRO)在Python中的工作原理十分重要。**

---

#### Python2中的旧式类与super

Python2中的super中只适用于新式类（需继承object)，同时Python2还保留着旧式类，Python3不再保留旧式类，虽然显示的继承object在Python3中是冗余的，但是如果想要垮版本兼容，继承object是一个好的做法，不然会有难以诊断的问题。

---

#### Python的方法解析顺序

**！！！预留位。。。。。**



MRO,C3算法。。。

类的\_\_MRO\_\_可以查看类的解析顺序

---

#### 使用super易犯的错误

在多重继承情况下，使用super是非常危险的，在Python中，\_\_init\_\_不会被隐式的调用。

**1. 混用super与显式类调用**

```python
class A:
    def __init__(self):
        print("A",end=" ")
        super().__init__()

class B:
    def __init__(self):
        print("B",end=" ")
        super().__init__()

class C(A,B):
    def __init__(self):
        print("C", end=" ")
        A.__init__(self)#显式的类调用与super混用
        B.__init__(self)#显式的类调用与super混用

>>>print("MRO":[x.__name for x in C.__MRO__])
>>>MRO:['C','A','B','object']
>>>C()
>>>C A B B #B被输出了2次
```

上面例子输出2次的原因是，C的实例调用了A.\_\_init\_\_(self),而A.\_\_init\_\_(self)中又调用了B.\_\_init\_\_(self)，所以造成了输出2次的结果。

如果需要对一个经三方类进行子类化，了好**总是**查看其内部代码主MRO中其他类的内部代码。

**2. 不同各类的参数**

使用super在初始化\_\_init\_\___中参数传递，如果没有相同的签名，一个类无法调用其基类的\_\_init()\_\_代码。

```python
class CommonBase:
    def __init__(self):
        print("CommonBase", end=" ")
        super(CommonBase, self).__init__()


class Base1(CommonBase):
    def __init__(self):
        print("Base1")
        super(Base1, self).__init__()


class Base2(CommonBase):
    def __init__(self):
        print("BASE2")
        super(Base2, self).__init__()


class MyClass(Base1, Base2):
    def __init__(self,arg):
        print("my base{}".format(arg))
        super(MyClass, self).__init__(arg)

c = MyClass(10)
>>>TypeError: __init__() takes 1 positional argument but 2 were given#报错
！！！如果MyClass中的super不要传参，去掉arg，可以正常使用
```

PS:这节中说解决这个问题，要么所有基类中添加*args与**kwargs魔法包装，但是这样会使代码变的脆弱，因为任何参数都可以传入并通过，还有一种可以在MyClass中显式调用特定类的init,但是这样又会发生super与显式调用混用的问题。 

**但是只要MyClass中的super不传入参数args，就没有影响**，对于书上这块表示疑惑！！！

---

#### 最佳实践

* 应该避免多重继承，使用14章节中的一些设计模式为代替
* super的使用必须一致，在类的层次结构中，要么全部使用super,要么全部不用
* 如果代码的使用范围包括Python2,那么在Python3中应该显式的集成object。
* 调用父类时必须查看类的层次结构（\_\_mro\_\_或mro())，为了避免出现任何问题，每次调用父类时，必须快速查看有关的MRO











