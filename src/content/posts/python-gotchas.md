---
  title: python gotchas
  pubDate: 2024-08-20
  categories: []
  description: 'A couple of annoying python footguns I've stumbled upon'
---
Every language has a couple of footguns that you will eventually accidentally trigger. And while I love Python, it shows it's age in some really annoying ways. In the following code samples try guessing what the output will be.


```python
word = "hello",
print(word)
print(word.to_upper())
```

    ('hello',)



    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    Cell In[2], line 3
          1 word = "hello",
          2 print(word)
    ----> 3 print(word.to_upper())


    AttributeError: 'tuple' object has no attribute 'to_upper'


This happens because Python parses a tuple automatically based on the comma, and if there is no value there it assumes it's a None value. In other languages, you always need to use parentheses to create a tuple.

It does allow for some convenient syntax, like `return a, b` instead of `return (a, b)`, but after having spent an hour debugging a bug caused by this, the conveniance is not worth it.

For the next one I've run into this so many times. It occurs whenever you have a class attribute that has the same name as a class function.


```python
class Hello:
    hello: str
    def __init__(self):
        self.hello = "Hello World"
    def hello(self):
        print(self.hello)

h = Hello()
h.hello()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[5], line 9
          6         print(self.hello)
          8 h = Hello()
    ----> 9 h.hello()


    TypeError: 'str' object is not callable


This occurs because Python parses every function call in the following way. `hello()` is first parsed as the attribute `hello`. It then calls the item retrieved from the attribute, which normally is the function you have defined. But since the class attribute is also called `hello`, and it was defined last, it will be retrieved instead of the function. Note that order really matters here. The following is even weirder. Since it always retrieves the function first. Thus it will call the function, but when it prints `self.hello` it will print the function object.


```python
class Hello:
    hello: str = "Hello World"
    def hello(self):
        print(self.hello)

h = Hello()
h.hello()
```

    <bound method Hello.hello of <__main__.Hello object at 0x7107c1f3c590>>

