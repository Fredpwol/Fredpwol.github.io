---
title: Python Decorators Demystified
date: 2020-07-07 10:00:00 +07:00
tags: [python, programming]
description: An in-depth explanation of pythons decorators
---

So what are decorators? Well basically decorators are function that takes another function and wraps it. You may have seen a decorator if you've used any web development framework be it Flask or Django in Flask it looks like this.
```python
@app.route('/home/')
def index():
    return render_template('index.html')
```

# Functions as first-class objects

To go in depth on how decorators work we need to understand how functions in python behave. In python functions are treated as first-class objects, what that means is that functions behave like normal python objects like `list, sets, integers, dict` e.t.c. and can be passed to variables and stored in containers such as list and also have attributes some example.

```python
def func(arg):
    return arg * 2

a = func
print(a(6))
print(func.__name__)
```
<b>Output</b>:
```shell
12
func
```
In the above code we created a function called `func` which returns the double of any number passed in, we then assigned the variable `a` to the function which means the variable `a` points to the address of the func object in memory. So calling `a(6)` will be the same as calling `func(6)`. The last line calls the `__name__` attribute of the function which returns the name of the function.

# Inner Functions

Functions in python can be defined in another function and can be accessed only inside the outer function. But to access the inner function in the global scope we return it and pass it to a variable in the global scope which will be used as reference to it.
```python
def some_function():

    def other_function(arg):
        if arg % 2 == 0: return True
        return False

    return other_function

ref = some_function()

print(ref(16))
```
<b>Output:</b>
```shell
True
```
As seen above the `other_function` is been passed to the `ref` variable when the `some_function` function is called. But if the `other_function` is called outside the function scope it will raise an error, so to access the `other_function` outside the `some_function` scope we return it and pass it to a variable which reference the function in memory.

# Use Case

Let begin to see how all we've learned so far about functions is going to build up to decorators and an actual use case of how it's used in the real world. So we are going to create a [factorial](https://www.mathsisfun.com/numbers/factorial.html) function and time it to see how long it takes on different inputs.

```python
import time
def factorial(n):
    res = 1
    while n > 0:
        res *= n
        n -= 1
    return res


start = time.time()
factorial(2000)
finish = time.time() - start
print("Execution factorial(%d) finished in %5f(s)"%(2000, finish)) 

start = time.time()
factorial(50000)
finish = time.time() - start
print("Execution factorial(%d) finished in %5f(s)"%(50000, finish)) 

start = time.time()
factorial(100000)
finish = time.time() - start
print("Execution factorial(%d) finished in %5f(s)"%(100000, finish))
```
<b>Output:</b>
```
Execution factorial(2000) finished in 0.015591(s)
Execution factorial(50000) finished in 2.854175(s)
Execution factorial(100000) finished in 15.139478(s)
```

The factorial function is passed different input and the time elapsed in calculating the output is what we care about. So the run time got printed out, but this code seems a lot messy, and we are not abiding by the DRY (Do not repeat yourself) principle. So to make this cleaner we use what we learned above about inner functions and utilize it for our goal. 

```python
def timer(func):
    
    def wrapper(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        finish = time.time() - start
        print("Execution %s(%s) finished in %5f(s)"%(func.__name__,args[0], finish))

    return wrapper
```

Ok there's a lot to unpack here but I'll make sure to break it down bits by bits. So basically the `timer` function takes another function as an argument as `func` and the `wrapper` function is an inner function that takes any amount of positional and keyword arguments, it's common to see the wrapper function written with `*args` and `**kwargs` more on positional and keyword arguments here: https://realpython.com/python-kwargs-and-args/. The use of `args and kwargs` in wrapper is because we want to be able to use the decorator on different functions with varying number of arguments so in order to handle that we use `args and kwargs`. We create our start timer and call the function passed to the `timer` function with the arguments passed to the wrapper after that we print the time elapsed for calling the function, then we return the wrapper function. Here's how to use the timer function.

```python
factorial = timer(factorial)
factorial(2000)
factorial(50000)
factorial(100000)
```
# Output:
```
Execution factorial(2000) finished in 0.000000(s)
Execution factorial(50000) finished in 2.691417(s)
Execution factorial(100000) finished in 14.838543(s)
```

We called the timer function and pass it the factorial function as argument and the wrapper function will be returned and passed to the factorial variable so the factorial is now a reference to the wrapper function and when executing `factorial(n)` is the same as executing `wrapper(n)`. The reason we pass in the original `factorial` function to the timer is for it to be available only in the local scope as `func` which is accessible to `wrapper` since its in the `timer` function. So when we call the factorial of any number we are calling the wrapper function and accessing the original factorial function pass the values to it and record its runtime and printing it out.

# Python Syntax Sugar

So what we built is a decorator in a sense, but to make this more pythonic, python have a syntax to make decorators easier to write using the `@` sign. The decorator is put above the function it wants to wrap and that's basically it.
```python
@timer
def factorial(n):
    res = 1
    while n > 0:
        res *= n
        n -= 1
    return res


factorial(2000)
factorial(50000)
factorial(100000)
```

It works the same as previous just more clean and pythonic and now the timer can be used for any function and that's why we needed to add `args and kwargs` to the wrapper because we don't know how many arguments may be passed in, as seen below we created a function that takes two parameters and the wrapper gets the value and pass it to the func.
```python
@timer
def waist_time(s,t):
    for _ in range(t):
        time.sleep(s)
```
<b>Output</b>:
```
Execution waist_time(1) finished in 10.009515(s)
```

# Some Constraints

There's a caveat when using decorators the function which is being wrapped by the decorator attributes will be changed to the wrapper function attributes as seen below.

```python
print(waist_time.__name__)
```
<b>Output:</b>
```
wrapper
```

This may not be a big deal to you or even change the functionality of your code, but its necessary to retain the function name for debugging purpose, so to retain the properties of the original function we wrap, to do this we use the `wrap` function in the `functools` module and decorate the wrapper function with it like this.

```python
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        finish = time.time() - start
        print("Execution %s(%s) finished in %5f(s)"%(func.__name__,args[0], finish))

    return wrapper
```

So running the defining the decorator like this and retains the attribute of the wrapped function and calling the `__name__` attribute it returns the function name.

# Nested Decorators

A function can be decorated with more than one decorator and the top decorator wraps the one below and the one below wraps what's below it.

```python
def pad_0(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        a = func()
        res = '0' + a + '0'
        return res
    return wrapper

def pad_5(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        a = func()
        res = '5' + a + '5'
        return res
    return wrapper

@pad_5
@pad_0
def greet(name):
    return "Hello %s"%(name)

print(greet("sarah"))
```
<b>Output:</b>
```
50Hello sarah05
```

As seen above the `pad_0` function wraps the `greet` function output then the `pad_5` function wraps the `pad_0` function and returns the output. More decorators can be stacked on top, but it begins to get difficult to manage or debug the function so you'll mostly use 3 to 1 decorators. And another useful thing to note is that the order of how the decorators are staked matters and changes the output take a look at this.

```python
@pad_0
@pad_5
def greet(name):
    return "Hello %s"%(name)

print(greet("sarah"))
```
<b>Output:</b>
```
05Hello sarah50
```

As you can see the output changes if the order of the decorator is changed slightly so it's a good thing to note when using stacked decorators.

# Decorator Argument

Decorators can be passed in arguments like normal function but doing this requires two or more nested functions in the decorator.
some example is `@app.route('/home/')`, here '/home/' is our argument passed to the route decorator, Here's how to create one.

```python
def repeat_n_times(n):
    def dec_wrapper(func):
        def wrapper(*args, **kwargs):
            for _ in range(n):
                print(func(*args, **kwargs))
        return wrapper
    return dec_wrapper

@repeat_n_times(5)
def say_hello():
    return "hello world!"
say_hello()
```
<b>Output:</b>
```
hello world!
hello world!
hello world!
hello world!
hello world!
```

# Advanced Use Case

One advanced use case of decorators are for memoization used in [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming), basically what the memoization is trying to do is reduce redundant function calls to improve speed of a program by storing computed output and access them in the future instead of re-computing it. so an example will be calculating [Fibonacci sequence](https://www.mathsisfun.com/numbers/fibonacci-sequence.html). An understanding of Fibonacci sequence and how to write a recursive algorithm to solve it in python is required, so you can check this out https://www.edureka.co/blog/python-fibonacci-series/ or follow along.

```python
def memoize_fibb(func):
    
    @functools.wraps(func)
    def wrapper(num):
        if len(wrapper.memo) == 0:
            wrapper.memo = [None] * (num+1)
        elif wrapper.memo[num]:
            return wrapper.memo[num]
        wrapper.memo[num] = func(num)
        return wrapper.memo[num]
        
    wrapper.memo = []
    return wrapper

@memoize_fibb
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(13))
```

The `wrapper.memo` is an attribute of the wrapper function and its a list which is initialized with n+1 values of none on the first call, then we check if the nth value is something other than `None` then we return the value in the list else we calculate it and store its value in the list on every function call. 
