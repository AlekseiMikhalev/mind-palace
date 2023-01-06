This section contains knowledge about asynchronous programming in Python

**Sources**: 

- Books: 

    - [Advanced Python Programming](https://www.packtpub.com/product/advanced-python-programming/9781838551216?utm_source=github&utm_medium=repository&utm_campaign=)

_Asynchronous programming_ is a way of dealing with slow and unpredictable resources. Rather than waiting idly for resources to become available, asynchronous programs can
handle multiple resources concurrently and efficiently. Programming in an asynchronous way can be challenging because it is necessary to deal with external requests that can arrive in any order, may take a variable amount of time, or may fail unpredictably.

_Concurrency_ is a way to implement a system that can deal with multiple requests at the same time. The idea is that we can move on and start handling other resources while we wait for a resource to become available. Concurrency works by splitting a task into smaller subtasks that can be executed out of order so that multiple tasks can be partially advanced without waiting for the previous tasks to finish.

```python
    import threading

    def wait_and_print_async(msg):
        def callback():
            print(msg)

        timer = threading.Timer(1.0, callback) #(1)
        timer.start()
```

1. a callback is simply a function that will be called when the timer expires. The strategy used by `threading.Timer` involves starting a new thread that can execute code in parallel.


Another library for concurrent code and reative programming is [RxPy](https://rxpy.readthedocs.io/en/latest/rationale.html)

Another idea to improve efficiency is to use multiprocessing. 

[TO BE CONTINUED...]