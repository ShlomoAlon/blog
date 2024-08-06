---
  title: mutable-nested-values
  pubDate: 2024-08-05
  categories: []
  description: 'Simple way of creating mutable values for a nested functions'
  ---

The traditional way of doing this is with a list like this.

```python
def recursive_function():
    mutable_values = [0]

    def inner_function():
        mutable_values[0] += 1
        print(mutable_values[0])

    return inner_function
```

however I find the following more readable and easier to understand. 
It takes advantage of the fact that all python objects are mutable and can be modified by nested functions.

```python
class MutableValues:
    pass

def recursive_function():
    mutable_values = MutableValues()
    mutable_values.value = 0

    def inner_function():
        mutable_values.value += 1
        print(mutable_values.value)

    return inner_function
```

  


  