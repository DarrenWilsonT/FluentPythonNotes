# 一摞Python风格的纸牌
## 特殊方法的实现和调用
```python
import collections
Card = collections.namedtuple('Card', ['rank', 'suit'])
class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()
    
    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]
    def __len__(self):
        return len(self._cards)
        
    def __getitem__(self, position):
        return self._cards[position]
```
- `collections.namedtuple`
  
    - 调用方式：`collections.namedtuple(typename, field_name, verbose=False, rename=False)`
    
    - 用途：用以构建只有少数属性但是没有方法的对象。
    
    - 示例：
    
      ```py
      import collections
      
      # 两种方法来给 namedtuple 定义方法名
      #User = collections.namedtuple('User', ['name', 'age', 'id'])
      User = collections.namedtuple('User', 'name age id')
      user = User('tester', '22', '464643123')
      
      print(user)
      ```
    
- `__getitem__`

    `[]`操作会调用对象的`__getitem__`方法，仅实现该方法便可以将对象变为可迭代的。

- `__len__`

    `len()`函数会调用对象的`__len__`方法。

- `random.choice`

    从一个序列中随机选出一个函数。

- 迭代：

    迭代通常是隐式的，譬如说一个集合类型没有实现`__contains__`方法，那么`in`运算符就会按顺序做一次迭代搜索。

```python
from math import hypot

class Vector:
    def __init__(self, x=0, y=0):
        self._x = x
        self._y = y
        
    # 将对象用字符串的形式表示出来，print()函数调用，在没有实现__str__方法时，__repr__可以作为替代
    def __repr__(self): 
        return 'Vector(%r, %r)' % (self._x, self._y)
    
    def __abs__(self): # abs()函数调用
        return hypot(self._x, self._y)
    
    def __bool__(self): # bool()函数调用，如果没有实现该方法，bool()函数将会尝试使用__len__作为替代
        return bool(abs(self))
    
    def __add__(self, other): # + 运算符调用
        x = self._x + other._x
        y = self._y + other._y
        return Vector(x, y)
    
    def __mul__(self, scalar): # * 运算符调用
        return Vector(self._x * scalar, self._y * scalar)
```

## 特殊方法一览

- 跟运算符无关的特殊方法

| 类别                    | 方法名                                                       |
| ----------------------- | ------------------------------------------------------------ |
| 字符串/字节序列表示形式 | `__repr__` `__str__` `__format__` `__bytes__`                |
| 数值转换                | `__abs__` `__bool__` `__complex__` `__int__` `__float__` `__hash__` `__index__` |
| 集合模拟                | `__len__` `__getitem__` `__setitem__` `__delitem__` `__contains__` |
| 迭代枚举                | `__iter__` `__reversed__` `__next__`                         |
| 可调用模拟              | `__call__`                                                   |
| 上下文管理              | `__enter__` `__exit__`                                       |
| 实例创建和销毁          | `__new__` `__init__` `__del__`                               |
| 属性管理                | `__getattr__` `__getattribute__` `__setattr__` `__delattr__` `__dir__` |
| 属性描述符              | `__get__` `__set__` `__delete__`                             |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |
|                         |                                                              |



- 属性访问拦截器：`__getattribute__`

  当该类的属性被访问时，会自动调用类的`__getattribute__`方法。默认的属性拦截器不进行任何操作直接返回。有需要时我们可以改写`__getattribute__`方法来实现相关功能，如查看权限，打印log等。

- 属性设置拦截器：`__setattr__`

  会拦截所有属性的的赋值语句。如果定义了这个方法，`obj.attr = value`就会变成`obj.__setattr__(self, attr, value)`。当在`__setattr__`方法内对属性进行赋值时，不可使用`self.attr = value`,因为他会再次调用`self.__setattr__(self, attr, value)`，则会形成无穷递归循环，最后导致堆栈溢出异常。应该通过对属性字典做索引运算来赋值任何实例属性，也就是使用`self.__dict__['name'] = value`。

  ```python
  class Tree(object):
      def __init__(self, name):
          self._name = name
          self._cate = "plant"
      
      def __getattribute__(self, attr):
          print("this is call getattribute "+attr)
          if attr.endswith('e'):
              return object.__getattribute__(self, attr)
          else:
              # 该返回方式错误，因为调用self.function还会再次调用self.__getattribute__方法，
              # 最终陷入死循环从而结束程序
              # return self.function
              
              return object.__getattribute__(self, attr)
      def __setattr__(self, attr, value):
          print("this is call setattr " + attr + value)
          self.__dict__[attr] = value
      
      def function(self):
          print("this is call function")
          return "this is return fo call function"
      
  
  tree = Tree("big tree")
  print("---------------------")
  tree._name = "small tree"
  print("---------------------")
  tree.function()
  ```

- 





















