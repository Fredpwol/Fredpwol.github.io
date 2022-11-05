---
title: An Introduction to concurrency in python
date: 2021-01-14 10:00:00 +07:00
tags: [programming, python]
description: A bried introduction to concurrency in python
---

So, what do we mean by running code concurrently? Well, a surface-level definition merely is running code in parallel. But shortly we would learn that is not always the case in some scenarios. Another question is why do we need to run our code concurrently? well first of it adds a performance boost to your code and it also makes working with your software smooth.
So in this post, I'm going to show you how to add concurrency to your python code to improve its performance but first, let's get a background of how python code runs normally.

## Introduction
The python Interpreter runs your code in a top to bottom manner, which means that all code depends on code that is above it to run before it executes. But that's not always the case, take for example this code snippet below.

```python
def do_something():
    time.sleep(2)
    print(10)

do_something()
print(20)
```

In the above code, the `do_something` function is called before we print 10, it sleeps for 2 sec which makes the interpreter idle then it then prints 10 before we get 20.
So imagine a scenario where we want to do stuff when our interpreter is idle in our case say we want to print 20 or do something else. one solution to that is creating `threads`.

## Threading
What is a `thread`? Well according to Wikipedia "A thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler, which is typically a part of the operating system". Woah that was a mouth full let me give a less technical definition based on my knowledge. A thread is a block of code that can run independently from each other and can be executed one at a time. In python only one thread can have access to the interpreter at a time this is because of the global interpreter lock (GIL) so it restricts threads from running in `parallel` which we would discuss more later but you can read more about the GIL [here](https://realpython.com/python-gil/). So how do threads work? well, when you hear concurrency the first thing that comes to mind is running code at the same time or in parallel, but that's not the case with threads they just give the illusion of running in parallel. Threads are stored in memory and only run when the interpreter is idle like in a case of sending a request to a server and waiting for a response, or opening a file and waiting for the stream, any IO task that requires waiting is an excellent example of when the interpreter goes idle. The image below illustrates how a synchronous or normal python code works.

![synchorous workflow](https://dev-to-uploads.s3.amazonaws.com/i/xcl7kdwwh0w8pqdz66bo.png)
If you observe the image above you'll see that our second I/O operation was dependent on the previous and waited for 2 seconds for the first operation to complete before it ran and used another 2 seconds before the operation ends. The whole operation took about 4 seconds and that's not good, Take a look at how using threading solves this.

![threading workflow](https://dev-to-uploads.s3.amazonaws.com/i/39gf0hbzn5urvcvj7yj0.png)
Like I said threads don't run in parallel, rather when another thread is idle the interpreter then executes the other thread so we save our self some waiting time and as you can see we get a performance boost, we got a runtime of 2.5 seconds.

### Threading in practice.
Well, this all sounds good but let's write some code to truly get practical use of threads. first I'll write some synchronous code to see how our example code behaves.
```python
import time

def do_something(sec):
	print("Sleeping for %d seconds..."%sec)
	time.sleep(sec)
	print("Done Sleeping for %d seconds"%sec)

start = time.time()
do_something(1)
do_something(2)
do_something(3)
do_something(4)
do_something(5)

print("Finished in %s seconds"%(time.time() - start))
```
Outputs
```
Sleeping for 1 seconds...
Done Sleeping for 1 seconds
Sleeping for 2 seconds...
Done Sleeping for 2 seconds
Sleeping for 3 seconds...
Done Sleeping for 3 seconds
Sleeping for 4 seconds...
Done Sleeping for 4 seconds
Sleeping for 5 seconds...
Done Sleeping for 5 seconds
Finished in 15.0119268894 seconds
```
We created a function that takes in an argument of sec then prints how many seconds it will sleep, sleeps then outputs Done sleeping for the value of sec. We then called our function with arguments from 1 to 5 inclusive. Our function will sleep for 1, 2, 3, 4, 5 seconds totaling 15 seconds before our code stop running. And if you observe the output you'll see that the function output is in other so 1 ran before 2 which also ran before 3 and so on.
Let look at how we can optimize our code with threads.

```python
import threading
import time

def do_something(sec):
	print("Sleeping for %d seconds..."%sec)
	time.sleep(sec)
	print("Done Sleeping for %d seconds"%sec)

start = time.time()
t1 = threading.Thread(target=do_something, args=(1,))
t2 = threading.Thread(target=do_something, args=(2,))
t3 = threading.Thread(target=do_something, args=(3,))
t4 = threading.Thread(target=do_something, args=(4,))
t5 = threading.Thread(target=do_something, args=(5,))

t1.start()
t2.start()
t3.start()
t4.start()
t5.start()
print("Finished in %s seconds"%(time.time() - start))
```
Output
```
Sleeping for 5 seconds...
Sleeping for 4 seconds...
Sleeping for 3 seconds...
Sleeping for 2 seconds...
Sleeping for 1 seconds...
Finished in 0.00273704528809 seconds
Done Sleeping for 1 seconds
Done Sleeping for 2 seconds
Done Sleeping for 3 seconds
Done Sleeping for 4 seconds
Done Sleeping for 5 seconds
```
I'll argue that the threading code seems bulkier but I assure you it will save you that runtime and also with loops you can reduce the amount of code written. so let's walk through the code above, we import the threading module this time it contains everything we need to run threads in python. our function is still the same but this time we create a `Thread` object and pass in our target function note that we don't use the parathesis when we pass it to the target argument that will automatically call the function so we just pass in the function object like this `target=do_something` instead of `target=do_something()`. so how do we pass in an argument to our target function? well the threading object takes in an argument of `args` which is for passing in positional arguments and also `kwargs` for keyword arguments. we used args to pass in our sec value like so `args=(2,)` in our example, args take in any iterable as an argument e.g list, tuple, range e.t.c. Optionally we can also use the `kwargs` argument which takes in a dictionary as its argument so in our example, it would look like this `kwargs={"sec":2}` those are the main arguments you need to know about the `Thread` class. We start each thread with the start method as we see all the threads are started in order but notice that we get our finish runtime before all our function call stops. This is intentional let me explain, so after all the threads are created with the start() method the interpreter doesn't stop It goes on and executes any synchronous code left it then when a thread finishes running it runs code after the sleep here we print "Done sleeping for n seconds". How do we let the interpreter wait for our threads to finish before we run any code? well, A solution is that we use the `join` method to block the interpreter till our threads finish running, our code will then look like this.
```python
import threading
import time

def do_something(sec):
	print("Sleeping for %d seconds..."%sec)
	time.sleep(sec)
	print("Done Sleeping for %d seconds"%sec)

start = time.time()
t1 = threading.Thread(target=do_something, args=(1,))
t2 = threading.Thread(target=do_something, args=(2,))
t3 = threading.Thread(target=do_something, args=(3,))
t4 = threading.Thread(target=do_something, args=(4,))
t5 = threading.Thread(target=do_something, args=(5,))

t1.start()
t2.start()
t3.start()
t4.start()
t5.start()

t1.join()
t2.join()
t3.join()
t4.join()
t5.join()
print("Finished in %s seconds"%(time.time() - start))
```
Outputs
```
Sleeping for 1 seconds...
Sleeping for 2 seconds...
Sleeping for 3 seconds...
Sleeping for 4 seconds...
Sleeping for 5 seconds...
Done Sleeping for 1 seconds
Done Sleeping for 2 seconds
Done Sleeping for 3 seconds
Done Sleeping for 4 seconds
Done Sleeping for 5 seconds
Finished in 5.00275301933 seconds
```
As you can see the total time of execution is output now after all the threads are completed. so the join method blocks the interpreter from doing anything until the thread it's called on runs till completion. You can try experimenting with this code for a bit, try rearranging the other of the start calls and the join calls, see what happens. It will also help you understand how everything works intuitively. We've covered threading now we would try another method which is multiprocessing.

## Multiprocessing
So while threading doesn't enable you to run your code in parallel multiprocessing does that. Using multiprocessing enables you to put your CPU to good use. Modern CPUs come with multiple cores a typical modern-day PC comes with an average of 4 cores, but because of the python GIL we can't use all the cores at once, we use only one core per operation. Multiprocessing lets us make use of all our cores by creating processes that run on each core at the same time, again it is parallel. Multiprocessing shines best when doing CPU-bound tasks so in this example instead of using our dummy `do_something` function I'll create a function that finds all perfect squares in a given range so our code will look like this.
```python

import time
import math

def find_perfect_squares(n):
	print(f"Finding perfect squares for values from 1 to {n}")
	res = []
	for i in range(1, n+1):
		if math.sqrt(i) in range(1, n+1):
			res.append(i)
	print(f"Found {len(res)} values in range of {n}")
	return res


start = time.time()
find_perfect_squares(1000)
find_perfect_squares(2000)
find_perfect_squares(5000)
find_perfect_squares(3000)
find_perfect_squares(100)
finished = time.time() - start 

print("Finished in %s seconds"%(finished))
```
Output
```
Finding perfect squares for values from 1 to 1000
Found 31 values in range of 1000
Finding perfect squares for values from 1 to 2000
Found 44 values in range of 2000
Finding perfect squares for values from 1 to 5000
Found 70 values in range of 5000
Finding perfect squares for values from 1 to 3000
Found 54 values in range of 3000
Finding perfect squares for values from 1 to 100
Found 10 values in range of 100
Finished in 8.968952894210815 seconds
```
If you observe you'll notice that our function here is highly computational and has a time complexity of `O(n^2)` which isn't the best. the total computation of our function calls is approximately 9 seconds. and some less computational calls e.g for range 100 has to wait for other highly computational calls like the range of 5000, we can run all of these in their individual process to boost our performance. The python multiprocessing API is similar to that of threads so learning both won't be a problem, let take a look at how to optimize this using multiprocessing.
```python
import multiprocessing
import time
import math

def find_perfect_squares(n):
	print(f"Finding perfect squares for values from 1 to {n}")
	res = []
	for i in range(1, n+1):
		if math.sqrt(i) in range(1, n+1):
			res.append(i)
	print(f"Found {len(res)} values in range of {n}")
	return res


if __name__ == '__main__':
		
	start = time.time()
	args = [1000, 2000, 5000, 3000, 100]
	processes = []
	
	for val in args:
		p = multiprocessing.Process(target=find_perfect_squares, args=(val, ))
		processes.append(p)

	for p in processes:
		p.start()

	for p in processes:
		p.join()
	finished = time.time() - start 

	print("Finished in %s seconds"%(finished))
```
Output
```
Finding perfect squares for values from 1 to 100
Found 10 values in range of 100
Finding perfect squares for values from 1 to 1000
Found 31 values in range of 1000
Finding perfect squares for values from 1 to 2000
Found 44 values in range of 2000
Finding perfect squares for values from 1 to 3000
Found 54 values in range of 3000
Finding perfect squares for values from 1 to 5000
Found 70 values in range of 5000
Finished in 3.448301315307617 seconds
```
To use multiprocessing we import the multiprocessing module as we did for threading. So I tried to clean my code this time by using for-loops to create, start, and join the process. I also stored all my process object in a list. we used the join method the same as before to block the interpreter until the process is completed. Also if we notice the output we see that the other of the output isn't the same as the input here the first to finish execution is outputted followed by the second till the least which has the slowest execution time.
> Make sure you create a process inside the `if __name__ == "__main__"` block. See the [docs](https://docs.python.org/2/library/multiprocessing.html#programming-guidelines) for more details.

There's an easier way to create processes in python, we use the Pool class to create a pool of processes and map through them like so.

```python
from multiprocessing import Pool
#snippet
if __name__ == "__main__":
    #snippet
    p = Pool(processes=5)
    p.map(find_perfect_squares, [1000, 2000, 5000, 3000, 100])
```

We can also get the returned value of the function call in our process by getting the returned value of the map method which is a list of our returned values.
```python
#snippet
pool = p.map(find_perfect_squares, [1000, 2000, 5000, 3000, 100])
print(pool)
```
Output
```
[[1, 4, 9, 16, 25, 36, 49, 64, 81, 100, 121, 144, 169, 196, 225, 256, 289, 324, 361, 400, 441, 484, 529, 576, 625, 676, 729, 784, 841, 900, 961], [1, 4, 9, 16, 25, 36, 49, 64, 81, 100, 121, 144, 1]...]
```
This is a trimmed output because of the size, the index of the returned value is the same as the index of the input so pool[0] will be the returned value of `find_perfect_squares(1000)` and so on.

## When to use which one
So like I've said before threading performs well on IO-bound like sending requests to a server, working with files, and any networking task. it doesn't work well with computational or CPU bound task and sometimes performs worst on those tasks. while multiprocessing can be used for IO tasks it also has its own drawbacks one is that every process memory scope is different and can't access the other. so it limits us to what we can do with it and it's why we mostly use it for computation. and also most times you won't need to use them unless you really need to optimize your code. but what to use depends on the type of your problem.

## Further Reading Resources
- https://realpython.com/python-concurrency/
- https://docs.python.org/3/library/concurrent.futures.html
- https://en.wikipedia.org/wiki/Concurrency_(computer_science)

