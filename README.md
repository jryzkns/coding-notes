# coding-notes

This is a collection of notes and ideas for programming problems and writing python. They do not come in any specific order.

## Pythonic way of working with both indices and values in an iterable

In python, people often write code such as the following:

```py
def linear_search(ls : List[int], target : int) -> Maybe[int]: 
    '''given a list and a target value, return the index if it is found'''
    for i in range(len(ls)):
        if ls[i] == target:
            return i
        print(f"at index {i}, the value {ls[i]} != {target}")
    return None
```

The `range(len())` pattern is not exactly pythonic and comes from a C way of doing things. A much better alternative is the following using `enumerate`:

```py
def linear_search(ls : List[int], target : int) -> Maybe[int]: 
    '''given a list and a target value, return the index if it is found'''
    for i, val in enumerate(ls):
        if val == target:
            return i
        print(f"at index {i}, the value {val} != {target}")
    return None
```

The reason for this is two-fold:
 - `enumerate` allows you to gather the indices while still iterating through the iterable, making both the index and value available without having to do indexing
 - the lack of indexing actually means faster code, as in the `range(len())` example `ls[i]` is called twice per iteration, meaning two memory dereferences, but `val` in the `enumerate` version is a local variable, meaning it was retrieved from memory once, and it's available for use as local memory.

## Triangularization

When making pairwise comparisons within a collection, one could tabulate the results in a square matrix. For example, given a binary operator `F` and a list of three people `['Vayne', 'Cait', 'Ashe']`, if we apply `F` on all pairs, we can come up with this matrix:

| F     | Vayne | Cait | Ashe |
|-------|-------|------|------|
| Vayne |  vFv  | vFc  | vFa  |
| Cait  |  cFv  | cFc  | cFa  |
| Ashe  |  aFv  | aFc  | aFa  |

depending on the properties of `F`, we can apply some optimizations to working with this operator and pairwise comparisons. Let's say `F` is the distance between two people. This operator has a few properties that we can exploit:

 - the distance between a person and oneself is always 0
 - the distance between `a` and `b` is the same as the distance between `b` and `a`

We are most interested in the second property, which is symmetricity. Knowing these properties, our table can be a lot smaller:

| F     | Vayne | Cait | Ashe |
|-------|-------|------|------|
| Vayne |  0    |      |      |
| Cait  |  cFv  | 0    |      |
| Ashe  |  aFv  | aFc  | 0    |

Suppose the task is to find the pair of people who are the farthest away from each other, then we only have to check `cFv`, `aFv`, and `aFc`, instead of all 9 possibilities. For this class of problems of comparing things in a pairwise fashion, we still have a O(n^2) complexity in terms of work completed, but it will shave down the running time by half of doing every comparison.

As a more engaged example, let's consider the problem of finding the biggest sum attainable from two *different* numbers in a list:

```py
def maxPairSum(ls: List[int]) -> int:
    max_pair = -1
    for i, val in enumerate(ls):
        for other_val in ls[i + 1:]:
            max_pair = max(val + other_val, max_pair)
    return max_pair
```

This is in comparison to the slightly simpler code that runs 2x slower:

```py
def maxPairSum(ls: List[int]) -> int:
    max_pair = -1
    for val in ls:
        for other_val in ls:
            max_pair = max(val + other_val, max_pair)
    return max_pair
```

## Setting up a simple memory structure for iterables

Often times when traversing an iterable, we would want to keep track of things that we have already seen. This will help us reduce the amount of work needed to be done at the tradeoff of using some memory. Consider the following example of checking if all the elements in a list are unique:

```py
def is_unique(ls: List[int]) -> bool:
    for idx, elem in enumerate(ls):
        if elem in ls[idx + 1:]:
            return False
    return True
```

In the above example we repeated ask if an element is in the region that we have already seen before `if elem in ls[idx + 1:]`. Note that the `in` operation here is O(n) time. As we are iterating through the entire list and checking for membership each time, this makes this algorithm overall a O(n^2) time operation. In terms of memory usage, it is O(1).

There is a lot of unnecessary work done where the same index in the list has to be looked at multiple times. Suppose we introduce a memory structure, we can then save on the amount of time needed at the cost of using some memory:

```py
def is_unique(ls: List[int]) -> bool:
    seen = set()
    for element in ls:
        if element in seen:
            return False
        seen.add(element)
    return True
```

Now the `in` operation is on average O(1) time. Given we are doing this for each element in the list, we now have a O(n) time algorithm. However, since we are saving values to look up, it has O(n) memory usage

## Pythonic way of bundling data together and applying them

One of the best functionalities in python is its ability to unpack iterables with the `*` operator. Suppose there is a function with the following signature:

```py
def add3(a:int, b:int, c:int) -> int:
    return a + b + c
```

To call this function, we would use something to the effect of `add3(1,2,3)`. However, if our arguments are bundled together in a tuple (and multiple pieces of data in python are often bundled together in tuples), then we would have to do something like this `add3(triple[0], triple[1], triple[2])`. This is easily replacable with the equivalent `add3(*triple)`. In short, when calling a function, the `*` operators allows an iterable to be unpacked into a list of arguments. This is often seen in documentation where a function takes in `*args`

Similarly with dictionaries, some functions may take in `**kwargs` which are named arguments. A prominent example would be working with the subprocess module and calling commands:

```py
import subprocess
proc = subprocess.run(['uname'], stdout = subprocess.PIPE, stderr = subproces.PIPE)
```
Here, `stdout` and `stderr` are keyword arguments to `subprocess.run`. It is lengthy and if we are calling `subprocess.run` multiple times, is quickly becomes an annoyance. In order to make this cleaner to look at, we can save the kwargs into its own dictionary and unpack it into the function calls:

```py
pipe = {'stdout':subprocess.PIPE, 'stderr':subprocess.PIPE}
proc = subprocess.run(['uname'], **pipe)
```

## Pythonic way of iterating by "columns"

Consider the problem of simultaneously iterating `n` iterables at the same time. This can take shape multiple forms but the most prominent examples are trying to find the longest common prefix among `n` strings, or transposing a `mxn` matrix. In both cases discussed above, we are generally "iterating by columns" rather by rows.

Whenever the idea of "look at the first of everything then look at the next of everything" pops up, using the `zip` function should immediately come to mind. We proceed with some examples:

```python
fruits = ('apple', 'banana', 'dragonfruit')
approvals = (0.85, 0.6, 0.9)
for fruit_name, rating in zip(fruits, approvals):
    print(f"The rating for {fruit} is {rating * 100}%")
```

The above snippet of code *simultaneously* iterates both of the `fruits` and `approvals` iterables, in essence the `zip` function literally zips together two iterables.

To see the "iterating by columns" idea explicitly, we now look at how to transpose a `mxn` matrix as represented by a list of lists in python:
 
``` py
>>> M = [[1,2], [3,4]]
>>> for row in M:
...     print(row)
...
[1, 2]
[3, 4]
# note this means to take each of the rows in M and traverse it one by one
>>> for col in zip(*M):
...     print(col)
...
(1, 3)
(2, 4)
>>> for col in map(list, zip(*M)):
...     print(col)
...
[1, 3]
[2, 4]
# map is a lazily evaluated, so we have to convert to list to get results
>>> list(map(list, zip(*M)))
[[1, 3], [2, 4]]
```

## Using a data structure to imitate another data structure

Consider a stack data structure, which is not natively implemented in most languages. Suppose we want to use a stack, we would begin by looking at its interface, or what it should allow us to do, which are the following:
 - `peek`: look at the topmost element on the stack
 - `push`: adding an element to the top of the stack
 - `pop`: removing an element from the top of the stack

In Python, despite having a stack implementation in `collections`, we shall look at how we can use a list to flex into a stack:

|stack|list|
|-|-|
|`stack.peek()`|`ls[-1]`|
|`stack.push`|`ls.append`|
|`stack.pop`|`ls.pop`|

Now consider a queue data structure, which is also not natively implemented in most languages. Suppose we want to use a queue, we would first take a look at its interface, which are the following:

 - `enqueue`: add item to the queue
 - `dequeue`: remove item from the queue
 - `front`: get front item of the queue
 - `rear`: get rear item of the queue

In Python, once again we will be using a list to mimic a queue despite an existing deque implementation in `collections`:

|queue|list|
|-|-|
|`queue.enqueue`|`ls.insert(0, elem)`|
|`queue.dequeue()`|`ls.pop()`|
|`queue.front()`|`ls[-1]`|
|`queue.rear()`|`ls[0]`|

A more non-trivial example would be working with a set with elements from a limited domain (ex. chars). Given an UTF-8 character is 1 byte large, we know that there are only a total of 256 possible characters. Instead of creating a set and adding characters to the set (O(logn) operation each time), we can opt for an array of 256 0's and 1's to achieve the same effect:

|set|array|
|-|-|
|`set.add`|`arr[] = 1`|
|`set.remove`|`arr[] = 0`|
|`set.lookup`|`arr[] == 1`|

In lower level languages such as C, working this way will often yielf faster solutions without too much sacrifice on the clarity of code.
