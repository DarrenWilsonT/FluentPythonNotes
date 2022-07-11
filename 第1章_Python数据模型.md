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

    迭代通常是隐式的，譬如说一个集合类型没有实现`__contains__`方法，那么in运算符就会按顺序做一次迭代搜索。































