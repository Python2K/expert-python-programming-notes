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

~~!!!super有个很重要的一点注意，其第二个参数是可选的，如果只提供了一个参数，那么将返回一个unbound类型，这一点在与classmethod一起使用时**特别有用**，以下为示例：~~

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

~~注意，零参数的super()也可以用于被@classmethod装饰器装饰的方法，无参数调用的super()被看作仅仅定义了第一个参数。~~

划线部分未通过测试，暂时记录。。。。。。

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

---

#### 高级属性访问模式

dir(object)可以列出所有属性方法

双下划线\_\_name不应该不应该使用，如果要私有属性，单下划线\_name是习惯做法。

---

#### 描述符

描述符(descriptor)允许自定义在引用一个对象的属性时应该完成的事情。

descriptor是复杂属性访问的基础，它的内部被用于实现property、方法、类方法、静态方法和super类型，它是一个**类**，定义了另外一个类的访问方式。

#### 对象属性控制

> 1. ***\_\_getattribute\_\_****是实例对象查找属性或者方法的入口，实例对象访问属性或方法时都需要调用到此方法。*
> 2. ***\_\_getattr\_\_(self, name)****可以用来在当用户试图访问一个根本不存在（或者暂时不存在）的属性时，来定义类的行为。当**\_\_getattribute\_\_**方法找不到属性时，最终会调用**\_\_getattr\_\_**方法。它可以用于捕捉错误的以及灵活地处理AttributeError。只有当试图访问不存在的属性时它才会被调用。
> 3. **\_\_setattr\_\_(self, name, value)****\_\_setattr\_\_**方法允许你自定义某个属性的赋值行为，不管这个属性存在与否，都可以对任意属性的任何变化都定义自己的规则。关于**\_\_setattr\_\_**有两点需要说明：第一，使用它时必须小心，不能写成类似**self.name = “Tom”**这样的形式，因为这样的赋值语句会调用**__setattr__**方法，这样会让其陷入无限递归；第二，你必须区分 ** **对象属性**和 **类属性** 这两个概念。
> 4. **\_\_delattr\_\_(self, name)**用于处理删除属性时的行为。和**\_\_setattr\_\_**方法要注意无限递归的问题，重写该方法时**不要有类似del self.name的写法**。

以上这几个对象属性控制的方法，具有通用性，也就是说，他控制的是一个对象的所有属性（是一般逻辑），并不能单独控制**单独某一个属性**的行为。

而**描述符**可以单独的抽离出一个属性对象，在属性对象中定义这个属性的查找、设定、删除行为，这个属性对象就是描述符。

---

描述符对象一般是作为其他类对象的属性则存在，描述符**类**基于3个特殊方法，这3个方法组成了descriptor protocol描述符协议。

* \_\_set_\_\(self,obj,type=None)：在设置属性时将调用这一方法;
* \_\_get\_\_(self,obj,value)：在读取属性时将使用这一方法（被称为getter);
* \_\_delete\_\_(self,obj)：对属性**调用**del时使用这一方法。

*同时实现了\_\__set_\_\(self,obj,type=None)与\_\_get\_\_(self,obj,value)被称为数据表述符(data descriptor)*

*只实现了\_\_get\_\_(self,obj,value)被称为非数据描述符(non-data descriptor)*

在每次属性查找中，这个协议的方法实际上由对象的特殊方法\_\_getattribute\_\_()调用（不要与\_\_getattr\_\_()弄混，后者用于在前者找不到属性的情况下，才会调用）每次通过instance.attribute或getattr(instance,'attribute')时候，都会隐式的调用\_\_getattribute\_\_(),它按下列顺序查找属性：

1. 验证该属性是否是实例的类对象的数据描述符
2. 如果不是，就查看该属性是否能在实例对象的\_\_dict\_\_中找到
3. 最后，查看该属性是否是实例的类对象的非数据描述符

Python官方示例文档

```python
class RevealAccess(object):
    """一个数据描述符，正常设定值并返回值，同时打印出记录访问的信息"""
    def __init__(self,initval=None,name="var"):
        self.val = initval
        self.name = name
    
    def __get__(self,obj,objtype):
        print("retrieving",self.name)
        return self.val
    
    def __set__(self,obj,val):
        print("Updating",self.name)
        self.val = val
        
class MyClass(object):
    x = RevealAccess(10,'var "x"')
    y = 5
    
>>>m = MyClass()
>>>m.x
retrieving var "x"
>>>print(m.x)
retrieving var "x"
10
>>>m.x = 20
Updating var "x"

```

上面这个例子，展示了描述符的作用。







