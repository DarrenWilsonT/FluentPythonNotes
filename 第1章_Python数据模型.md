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

### 跟运算符无关的特殊方法

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
| 跟类相关的服务          | `__prepare__` `__instancecheck__` `__subclasscheck__`        |



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

- [属性描述符](https://blog.csdn.net/Leafage_M/article/details/54960432)：`__get__` `__set__` `__delete__`

  ```python
  class Descriptor(object):  
      def __get__(self, obj, type=None):  
              return 'get', self, obj, type  
      def __set__(self, obj, val):  
          print 'set', self, obj, val  
      def __delete__(self, obj):  
          print 'delete', self, obj 
          
  class T(object):  
         d = Descriptor()  
  
  t = T()
  
  >>> t.d         #t.d，返回的实际是d.__get__(t, T)  
  ('get', <__main__.Descriptor object at 0x00CD9450>, <__main__.T object at 0x00CD0E50>, <class '__main__.T'>)  
  >>> T.d        #T.d，返回的实际是d.__get__(None, T)，所以obj的位置为None  
  ('get', <__main__.Descriptor object at 0x00CD9450>, None, <class '__main__.T'>)  
  >>> t.d = 'hello'   #在实例上对descriptor设置值。要注意的是，现在显示不是返回值，而是__set__方法中print语句输出的。  
  set <__main__.Descriptor object at 0x00CD9450> <__main__.T object at 0x00CD0E50> hello  
  >>> t.d         #可见，调用了Python调用了__set__方法，并没有改变t.d的值  
  ('get', <__main__.Descriptor object at 0x00CD9450>, <__main__.T object at 0x00CD0E50>, <class '__main__.T'>)  
  >>> T.d = 'hello'   #没有调用__set__方法  
  >>> T.d                #确实改变了T.d的值  
  'hello'  
  >>> t.d  #t.d的值也变了，这可以理解，按我们上面说的属性查找策略，t.d是从T.__dict__中得到的T.__dict__['d']的值是'hello'，t.d当然也是'hello'  
  'hello'
  ```

### 跟运算符相关的特殊方法

| 类别               | 方法名和对应的运算符                                         |
| ------------------ | ------------------------------------------------------------ |
| 一元运算符         | `__neg__` ：- ， `__pos__` : +, `__abs__`: `abs()`           |
| 众多比较运算符     | `__lt__` <、`__le__` <=、`__eq__ `==、`__ne__` !=、`__gt__` >、`__ge__` >= |
| 算术运算符         | `__add__` +、`__sub__ `-、`__mul__` *、`__truediv__` /、`__floordiv__` //、`__mod__` %、`__divmod__` `__divmod()`、`__pow__` ** 或`pow()`、`__round__` `round()` |
| 反向算是运算符     | `__radd__`、`__rsub__`、`__rmul__`、`__rtruediv__`、`__rfloordiv__`、`__rmod__`、`__rdivmod__` |
| 增量赋值算术运算符 | `__iadd__`、`__isub__`、`__imul__`、`__itruediv__`、`__ifloordiv__`、`__imod__`、`__ipow__` |

## 小结

- 通过特殊方法，我们可以让自定义数据类型可以表现得和内置类型一样，从而使得代码更加pythonic。
- Python 对象的一个基本要求就是它得有合理的字符串表示形式，我们可以通过 __repr__ 和 __str__ 来满足这个要求。前者方便我们调试和记录日志，后者则是给终端用户看的。这就是数据模型中存在特殊方法`__repr__` 和 `__str__` 的原因。





