## 1. 鏈接迭代器 (Chaining Iterables)
--- 

### 1.1 基本鏈接
`itertools.chain(*args)` 允許我們將多個迭代器連接成一個懶惰迭代器。

例如:
```python
from itertools import chain

iter1 = [1, 2, 3]
iter2 = 'abc'
iter3 = (4, 5, 6)

for item in chain(iter1, iter2, iter3):
    print(item)

# 輸出: 1 2 3 a b c 4 5 6
``` 

這相當於手動實現 :
```python
def manual_chain(*iters):
    for it in iters:
        yield from it

for item in manual_chain(iter1, iter2, iter3):
    print(item)
```

### 1.2 從可迭代對象中鏈接
`itertools.chain.from_iterable(it)` 允許我們從一個包含多個迭代器的可迭代對象中創建鏈接。

例如:
```python
iters = [[1, 2], 'ab', (3, 4)]
for item in chain.from_iterable(iters):
    print(item)

# 輸出: 1 2 a b 3 4
```
這相當於:
```python
def chain_lazy(it):
    for sub_it in it:
        yield from sub_it

for item in chain_lazy(iters):
    print(item)
```
--- 

## 2. 分流迭代器 (Teeing Iterables)

`itertools.tee(iterable, n)` 允許我們從一個迭代器創建多個獨立的迭代器。

例如:
```python
from itertools import tee

original = [1, 2, 3, 4]
iter1, iter2, iter3 = tee(original, 3)

print(list(iter1))  # [1, 2, 3, 4]
print(list(iter2))  # [1, 2, 3, 4]
print(list(iter3))  # [1, 2, 3, 4]
```
注意: 即使原始輸入是一個列表,`tee` 返回的也始終是懶惰迭代器。

---

## 3. 注意事項

- 鏈接操作是懶惰的,只在需要時才生成元素
- 分流操作創建獨立的迭代器,可以並行或多次使用
- 這些操作有助於高效處理大量數據或無限序列

## 4. 代碼示例

---


### Chaining and Teeing Iterators

Often we need to chain iterators / iterables together to behave like a single iterable.
We can think of this as analogous to sequence concatenation.
For example, suppose we have some generators producing squares:


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))
```

And we want to essentially iterate through all the values as if they were a single iterator.

We could do it this way:


```python
for gen in (l1, l2, l3):
    for item in gen:
        print(item)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

In fact, we could even create our own generator function to do this :

```python
def chain_iterables(*iterables):
    for iterable in iterables:
        yield from iterable
```


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

for item in chain_iterables(l1, l2, l3):
    print(item)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

But, a much simpler way is to use `chain` in the `itertools` module:


```python
from itertools import chain
```


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

for item in chain(l1, l2, l3):
    print(item)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

Note that `chain` expects a variable number of positional arguments, each of which should be an iterable.

It will not work if we pass it a single iterable:


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(lists):
    print(item)
```

    <generator object <genexpr> at 0x0000020AAFE06F10>
    <generator object <genexpr> at 0x0000020AAFDB43B8>
    <generator object <genexpr> at 0x0000020AAFE06FC0>
    

As you can see, it simply took our list and handed it back directly - there was nothing else to chain with:


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(lists):
    for i in item:
        print(i)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

Instead, we could use unpacking:


```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(*lists):
    print(item)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

Unpacking works with iterables in general, so even the following would work just fine:


```python
def squares():
    yield (i**2 for i in range(4))
    yield (i**2 for i in range(4, 8))
    yield (i**2 for i in range(8, 12))
```


```python
for item in chain(*squares()):
    print(item)
```

    0
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100
    121
    

But, unpacking is not lazy!! Here's a simple example that shows this, and why we have to be careful using unpacking if we want to preserve lazy evaluation:


```python
def squares():
    print('yielding 1st item')
    yield (i**2 for i in range(4))
    print('yielding 2nd item')
    yield (i**2 for i in range(4, 8))
    print('yielding 3rd item')
    yield (i**2 for i in range(8, 12))
```


```python
def read_values(*args):
    print('finised reading args')
```


```python
read_values(*squares())
```

    yielding 1st item
    yielding 2nd item
    yielding 3rd item
    finised reading args
    

Instead we can use an alternate "constructor" for chain, that takes a single iterable (of iterables) and lazily iterates through the outer iterable as well:


```python
c = chain.from_iterable(squares())
```


```python
for item in c:
    print(item)
```

    yielding 1st item
    0
    1
    4
    9
    yielding 2nd item
    16
    25
    36
    49
    yielding 3rd item
    64
    81
    100
    121
    

Note also, that we can easily reproduce the same behavior of either chain quite easily:


```python
def chain_(*args):
    for item in args:
        yield from item
```


```python
def chain_iter(iterable):
    for item in iterable:
        yield from item
```

And we can use those just as we saw before with `chain` and `chain.from_iterable`:


```python
c = chain_(*squares())
```

    yielding 1st item
    yielding 2nd item
    yielding 3rd item
    


```python
c = chain_iter(squares())
for item in c:
    print(item)
```

    yielding 1st item
    0
    1
    4
    9
    yielding 2nd item
    16
    25
    36
    49
    yielding 3rd item
    64
    81
    100
    121
    

### "Copying" an Iterator

Sometimes we may have an iterator that we want to use multiple times for some reason.

As we saw, iterators get exhausted, so simply making multiple references to the same iterator will not work - they will just point to the same iterator object.

What we would really like is a way to "copy" an iterator and use these copies independently of each other.

We can use `tee` in `itertools`:


```python
from itertools import tee
```


```python
def squares(n):
    for i in range(n):
        yield i**2
```


```python
gen = squares(10)
gen
```

    <generator object squares at 0x0000020AAFE067D8>

```python
iters = tee(squares(10), 3)
```

```python
iters
```

    (<itertools._tee at 0x20aafe6b608>,
     <itertools._tee at 0x20aafe6bdc8>,
     <itertools._tee at 0x20aafe6b448>)

```python
type(iters)
```

    tuple

As you can see `iters` is a **tuple** contains 3 iterators - let's put them into some variables and see what each one is:

```python
iter1, iter2, iter3 = iters
```


```python
next(iter1), next(iter1), next(iter1)
```

    (0, 1, 4)


```python
next(iter2), next(iter2)
```

    (0, 1)

```python
next(iter3)
```

    0


As you can see, `iter1`, `iter2`, and `iter3` are essentially three independent "copies" of our original iterator (`squares(10)`)

Note that this works for any iterable, so even sequence types such as lists:

```python
l = [1, 2, 3, 4]
```

```python
lists = tee(l, 2)
```

```python
lists[0]
```

    <itertools._tee at 0x20aafe6b688>

```python
lists[1]
```

    <itertools._tee at 0x20aafe6b048>

But you'll notice that the elements of `lists` are not lists themselves!

```python
list(lists[0])
```

    [1, 2, 3, 4]

```python
list(lists[0])
```

    []

As you can see, the elements returned by `tee` are actually `iterators` - even if we used an iterable such as a list to start off with!

```python
lists[1] is lists[1].__iter__()
```

    True

```python
'__next__' in dir(lists[1])
```

    True


Yep, the elements of `lists` are indeed iterators!


## 5. Conclusion


- Chaining is useful for combining multiple iterables into a single iteration.
- Teeing is helpful when you need multiple independent copies of an iterator.
- Both operations can be memory-efficient when used correctly, especially with lazy evaluation.
- The `itertools` module provides powerful tools for working with iterators and iterables in Python.
- 
---