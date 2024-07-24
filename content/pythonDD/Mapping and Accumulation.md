

## 1. Mapping

Mapping 是將一個可調用對象(函數)應用到可迭代對象的每個元素上。

### 1.1 map() 函數

```python
map(fn, iterable)
```

- `fn` 是一個接受單一參數的可調用對象
- 返回一個懶惰迭代器

例子:
```python
squared = map(lambda x: x**2, [1, 2, 3, 4])
list(squared)  # [1, 4, 9, 16]
```

### 1.2 生成器表達式

我們也可以使用生成器表達式來實現相同的功能:

```python
squared = (x**2 for x in [1, 2, 3, 4])
```

### 1.3 itertools.starmap()

`starmap` 類似於 `map`，但它會解包可迭代對象的每個子元素，並將其作為參數傳遞給映射函數。

```python
from itertools import starmap
import operator

l = [[1, 2], [3, 4]]
result = starmap(operator.mul, l)
list(result)  # [2, 12]
```

等效的生成器表達式:
```python
result = (operator.mul(*item) for item in l)
```

## 2. Accumulation

Accumulation 是將可迭代對象減少到單一值的過程。

### 2.1 內置累積函數

- `sum(iterable)`: 計算可迭代對象中所有元素的和
- `min(iterable)`: 返回可迭代對象中的最小元素
- `max(iterable)`: 返回可迭代對象中的最大元素

### 2.2 functools.reduce()

```python
from functools import reduce

reduce(fn, iterable, [initializer])
```

- `fn` 是一個接受兩個參數的函數
- 將 `fn` 累積應用於可迭代對象的元素

例子:
```python
l = [1, 2, 3, 4]
sum_result = reduce(lambda x, y: x + y, l)  # 10
product_result = reduce(lambda x, y: x * y, l)  # 24
```

### 2.3 itertools.accumulate()

```python
from itertools import accumulate

accumulate(iterable, fn)
```

- 類似於 `reduce`，但返回所有中間結果的懶惰迭代器
- `fn` 是可選的，默認為加法操作

例子:
```python
import operator
l = [1, 2, 3, 4]
acc = accumulate(l, operator.mul)
list(acc)  # [1, 2, 6, 24]
```

## 3. 比較 reduce 和 accumulate

- `reduce` 只返回最終結果
- `accumulate` 返回所有中間結果
- `reduce` 接受一個初始值，`accumulate` 不接受
- 參數順序不同:
  - `reduce(fn, iterable)`
  - `accumulate(iterable, fn)`

## 總結

Mapping 和 Accumulation 操作是處理可迭代對象的強大工具。Mapping 允許我們轉換數據，而 Accumulation 允許我們將數據聚合成單一結果或一系列中間結果。這些操作在數據處理和函數式編程中非常有用。



## 範例代碼

### Mapping and Reducing

#### *map* and *starmap*

You should already know the `map` and `reduce` built-in functions, so let's quickly review them:

The `map` function applies a given function (that takes a single argument) to an iterable of values and yields (lazily) the result of applying the function to each element of the iterable.

Let's see a simple example that calculates the square of values in an iterable:

```python
maps = map(lambda x: x**2, range(5))
```

```python
list(maps)
```

    [0, 1, 4, 9, 16]

Keep in mind that `map` returns an iterator, so it will become exhausted:

```python
list(maps)
```

    []

Of course, we can supply multiple values to a function by using an iterable of iterables (e.g. tuples) and unpacking the tuple in the function - but we still only use a single argument:

```python
def add(t):
    return t[0] + t[1]
```

```python
list(map(add, [(0,0), [1,1], range(2,4)]))
```

    [0, 2, 5]

Remember how we can unpack an iterable into separate positional arguments?

```python
def add(x, y):
    return x + y
```

```python
t = (2, 3)
add(*t)
```

    5

It would be nice if we could do that with the `map` function as well.

For example, it would be nice to do the following:

```python
list(map(add, [(0,0), (1,1), (2,2)]))
```

    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-8-3a2b56e62a8f> in <module>()
    ----> 1 list(map(add, [(0,0), (1,1), (2,2)]))
    

    TypeError: add() missing 1 required positional argument: 'y'


But of course that is not going to work, since `add` expects two arguments, and only a single one (the tuple) was provided.

This is where `starmap` comes in - it will essentially `*` each element of the iterable before passing it to the function defined in the map:

```python
from itertools import starmap
```

```python
list(starmap(add, [(0,0), (1,1), (2,2)]))
```

    [0, 2, 4]


#### Accumulation

You should already know the `sum` function - it simply calculates the sum of all the elements in an iterable:

```python
sum([10, 20, 30])
```

    60


It simply returns the final sum.

Sometimes we want to perform other operations than just summing up the values. Maybe we want to find the product of all the values in an iterable.

To do so, we would then use the `reduce` function available in the `functools` module. You should already be familiar with that function, but let's review it quickly.

The `reduce` function requires a `binary` function (a function that takes two arguments). It then applies that binary function to the first two elements of the iterable, obtains a result, then continues applying the binary function using the previous result and the next item in the iterable.

Optionally we can specify a seed value that is used as the 'first' element.

For example, to obtain the product of all values in an iterable:

```python
from functools import reduce
```

```python
reduce(lambda x, y: x*y, [1, 2, 3, 4])
```

    24

We can even specify a "start" value:

```python
reduce(lambda x, y: x*y, [1, 2, 3, 4], 10)
```

    240

You'll note that with both `sum` and `reduce`, only the final result is shown - none of the intermediate results are available.

Sometimes we want to see the intermediate results as well.

Let's see how we might try it with the `sum` function:|

```python
def sum_(iterable):
    it = iter(iterable)
    acc = next(it)
    yield acc
    for item in it:
        acc += item
        yield acc
```

And we can use it as follows:

```python
for item in sum_([10, 20, 30]):
    print(item)
```

    10
    30
    60
    

Of course, this is just going to work for a sum.

We may want the same functionality with arbitrary binary functions, just like `reduce` was more general than `sum`.

We could try doing it ourselves as follows:

```python
def running_reduce(fn, iterable, start=None):
    it = iter(iterable)
    if start is None:
        accumulator = next(it)
    else:
        accumulator = start
    yield accumulator
    
    for item in it:
        accumulator = fn(accumulator, item)
        yield accumulator
    
```

Let's try a running sum first.

We'll use the `operator` module instead of using lambdas.

```python
import operator
```

```python
list(running_reduce(operator.add, [10, 20, 30]))
```

    [10, 30, 60]

Now we can also use other binary operators, such as multiplication:

```python
list(running_reduce(operator.mul, [1, 2, 3, 4]))
```

    [1, 2, 6, 24]

And of course, we can even set a "start" value:

```python
list(running_reduce(operator.mul, [1, 2, 3, 4], 10))
```

    [10, 10, 20, 60, 240]


While this certainly works, we really don't need to code this ourselves - that's exactly what the `accumulate` function in `itertools` does for us.

The order of the arguments however is different, The iterable is defined first - that's because the binary function is optional, and defaults to addition if we don't specify it. Also it does not have a "start" value option. If you really need that feature, you could use the technique I just showed you.

```python
from itertools import accumulate
```

```python
list(accumulate([10, 20, 30]))
```

    [10, 30, 60]

We can find the running product of an iterable:

```python
list(accumulate([1, 2, 3, 4], operator.mul))
```

    [1, 2, 6, 24]


