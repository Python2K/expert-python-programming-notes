## 第三章 语法最佳实践—类级别以上

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

super是一个内置类，用于访问属于某个对象的超类的属性，看上去似乎是个函数，但是其他是一个内置类。

在定义类时，方法内部使用super可以简化super(),括号中可以不传任何参数，但是如果不在方法内部使用，必须给出参数，如下：

```python
anita = Sister()
>>>super(anita.__class__, anita).says()
```

***!!!***super有个很重要的一点注意，其第二个参数是可选的，如果只提供了一个参数，那么将返回一个unbound类型，这一点在与classmethod一起使用时**特别有用**，以下为示例：



