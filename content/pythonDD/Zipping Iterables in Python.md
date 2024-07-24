

## 1. `zip` 函數

`zip` 是Python的內置函數,用於將多個可迭代對象"壓縮"在一起。

### 1.1 基本用法

```python
zip(*iterables)
```

- 接受任意數量的可迭代對象作為參數
- 返回一個懶惰迭代器,生成包含各個可迭代對象元素的元組
- 基於最短的可迭代對象停止迭代

### 1.2 例子

```python
numbers = [1, 2, 3]
letters = ['a', 'b', 'c']
zipped = zip(numbers, letters)
list(zipped)  # [(1, 'a'), (2, 'b'), (3, 'c')]

# 不等長可迭代對象
numbers = [1, 2, 3, 4]
letters = ['a', 'b', 'c']
zipped = zip(numbers, letters)
list(zipped)  # [(1, 'a'), (2, 'b'), (3, 'c')]
```

### 1.3 特點

1. 懶惰評估: 只在需要時生成下一個元組
2. 基於最短可迭代對象: 一旦最短的可迭代對象耗盡,`zip`就會停止

## 2. `itertools.zip_longest`

當我們想基於最長的可迭代對象進行壓縮時,可以使用`zip_longest`。

### 2.1 基本用法

```python
from itertools import zip_longest

zip_longest(*iterables, fillvalue=None)
```

- 接受任意數量的可迭代對象作為參數
- `fillvalue`參數指定填充值(默認為`None`)
- 返回一個懶惰迭代器,生成包含各個可迭代對象元素的元組

### 2.2 例子

```python
from itertools import zip_longest

numbers = [1, 2, 3]
letters = ['a', 'b', 'c', 'd']
zipped = zip_longest(numbers, letters, fillvalue=0)
list(zipped)  # [(1, 'a'), (2, 'b'), (3, 'c'), (0, 'd')]

# 使用自定義填充值
zipped = zip_longest(numbers, letters, fillvalue='*')
list(zipped)  # [(1, 'a'), (2, 'b'), (3, 'c'), ('*', 'd')]
```

### 2.3 特點

1. 基於最長可迭代對象: 繼續迭代直到所有可迭代對象都耗盡
2. 填充值: 為較短的可迭代對象提供默認值
3. 懶惰評估: 只在需要時生成下一個元組

## 3. `zip` vs `zip_longest` 比較

| 特性 | `zip` | `zip_longest` |
|------|-------|---------------|
| 停止條件 | 最短可迭代對象耗盡 | 最長可迭代對象耗盡 |
| 處理不等長 | 忽略多餘元素 | 使用填充值 |
| 填充值 | 不適用 | 可自定義 |
| 內置/導入 | 內置函數 | 需從itertools導入 |

## 4. 應用場景

1. `zip`: 當你確定所有可迭代對象長度相等,或只關心最短可迭代對象的元素時使用
2. `zip_longest`: 當你需要處理所有可迭代對象的所有元素,並為較短的可迭代對象提供默認值時使用

## 5. 注意事項

- 兩個函數都返回迭代器,如果需要立即獲得所有結果,可以使用`list()`轉換
- 使用`zip_longest`時要小心選擇填充值,確保它不會與實際數據混淆
- 在處理大量數據時,這兩個函數的懶惰評估特性可以幫助節省內存


## 6.範例代碼

### Zipping

We've already used the `zip` function quite a bit.

It zips up two iterables and yields tuples containing elements from all iterables in "parallel". It is also lazy, and it will stop once the first iterable is exhausted.

Let's look at a simple example:

```python
l1 = [1, 2, 3, 4, 5]
l2 = [1, 2, 3, 4]
l3 = [1, 2, 3]
```

```python
list(zip(l1, l2, l3))
```

    [(1, 1, 1), (2, 2, 2), (3, 3, 3)]

As you can see, the shortest iterable we provided to the `zip` function had a length of 3 (so it reached the end of iteration first), and our output therefore only had 3 tuples in it.

Of course, this works with iterators and generators too:

```python
def integers(n):
    for i in range(n):
        yield i
        
def squares(n):
    for i in range(n):
        yield i**2
        
def cubes(n):
    for i in range(n):
        yield i**3
```

```python
iter1 = integers(6)
iter2 = squares(5)
iter3 = cubes(4)
```

```python
list(zip(iter1, iter2, iter3))
```

    [(0, 0, 0), (1, 1, 1), (2, 4, 8), (3, 9, 27)]


Sometimes we want to zip up iterables but completely iterate all the iterables, and not stop at the shortest. Of course, the problem is what to do with iterables that have been full iterated before the longest one has?

Simple, we just need to provide a default "filler" value.

And that's how the `zip_longest` function from `itertools` works:

```python
from itertools import zip_longest
```

```python
help(zip_longest)
```

    Help on class zip_longest in module itertools:
    
    class zip_longest(builtins.object)
     |  zip_longest(iter1 [,iter2 [...]], [fillvalue=None]) --> zip_longest object
     |  
     |  Return a zip_longest object whose .__next__() method returns a tuple where
     |  the i-th element comes from the i-th iterable argument.  The .__next__()
     |  method continues until the longest iterable in the argument sequence
     |  is exhausted and then it raises StopIteration.  When the shorter iterables
     |  are exhausted, the fillvalue is substituted in their place.  The fillvalue
     |  defaults to None or can be specified by a keyword argument.
     |  
     |  Methods defined here:
     |  
     |  __getattribute__(self, name, /)
     |      Return getattr(self, name).
     |  
     |  __iter__(self, /)
     |      Implement iter(self).
     |  
     |  __new__(*args, **kwargs) from builtins.type
     |      Create and return a new object.  See help(type) for accurate signature.
     |  
     |  __next__(self, /)
     |      Implement next(self).
     |  
     |  __reduce__(...)
     |      Return state information for pickling.
     |  
     |  __setstate__(...)
     |      Set state information for unpickling.
    
    

As you can see, we can only specify a single default value, this means that default will be used for any provided iterable once it has been fully iterated.

As expected, `zip_longest` yields its values - it is lazy.

Let's see an example:

```python
l1 = [1, 2, 3, 4, 5]
l2 = [1, 2, 3, 4]
l3 = [1, 2, 3]
```

```python
list(zip_longest(l1, l2, l3, fillvalue='N/A'))
```

    [(1, 1, 1), (2, 2, 2), (3, 3, 3), (4, 4, 'N/A'), (5, 'N/A', 'N/A')]


Of course, since this zips over the longest iterable, beware of using an infinite iterable!

You don't have to worry about this with the normal `zip` function as long as at least one of the iterables is finite:

```python
def squares():
    i = 0
    while True:
        yield i ** 2
        i += 1

def cubes():
    i = 0
    while True:
        yield i ** 3
        i += 1
```

Obviously `squares` produces an inifinite iterator. But we can still zip it with a finite iterable:

```python
iter1 = squares()
iter2 = cubes()
list(zip(range(10), iter1, iter2))
```

    [(0, 0, 0),
     (1, 1, 1),
     (2, 4, 8),
     (3, 9, 27),
     (4, 16, 64),
     (5, 25, 125),
     (6, 36, 216),
     (7, 49, 343),
     (8, 64, 512),
     (9, 81, 729)]

Don't try the same thing with `zip_longest`!
