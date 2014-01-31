========================
Python Memoize Decorator
========================

.. sectionauthor:: momo <mobeiheart@gmail.com>

.. feed-entry::
   :date: 2013-10-11

`A comparison between node.js and python <http://stackoverflow.com/questions/19308835/why-python-is-much-slower-than-node-js-on-recursion/19308975#19308975>`_, measures the time of running recursive fibonacci functions, the former is much faster than the latter, which may be the cause of v8 engine::

    def memoize(f):
        cache = {}
        def decorated_function(*args):
            if args in cache:
                return cache[args]
            else:
                cache[args] = f(*args)
                return cache[args]
        return decorated_function

But you can use memoize in python to speed up, its function form sets up a closure cached the values have been computed.


On the other hand, it also can be implemented by `class <https://wiki.python.org/moin/PythonDecoratorLibrary#Memoize>`_::

    import collections
    import functools

    class memoized(object):
       '''Decorator. Caches a function's return value each time it is called.
       If called later with the same arguments, the cached value is returned
       (not reevaluated).
       '''
       def __init__(self, func):
          self.func = func
          self.cache = {}
       def __call__(self, *args):
          if not isinstance(args, collections.Hashable):
             # uncacheable. a list, for instance.
             # better to not cache than blow up.
             return self.func(*args)
          if args in self.cache:
             return self.cache[args]
          else:
             value = self.func(*args)
             self.cache[args] = value
             return value
       def __repr__(self):
          '''Return the function's docstring.'''
          return self.func.__doc__
       def __get__(self, obj, objtype):
          '''Support instance methods.'''
          return functools.partial(self.__call__, obj)

    @memoized
    def fibonacci(n):
       "Return the nth fibonacci number."
       if n in (0, 1):
          return n
       return fibonacci(n-1) + fibonacci(n-2)

    print fibonacci(12)

The class has an instance variable of dict to store the values, the cache key **must** be hashable for using as an dict key, ``memoized_function([1, 2], [3, 4])`` will not be cached::

    class Fibonacci(object):

        @memoized
        def get_result(self, n):
            if n in (0, 1):
                return n
            return self.get_result(n-1) + self.get_result(n-2)
        # provided: memoized_get_result = memoized(get_result)

    f = Fibonacci()
    f.get_result(10)

The definition of ``__get__`` lets ``f.get_result(*args)`` be equal to ``f.memoized_get_result.__call__(f.memoized_get_result, *args)`` and makes it possible to decorate instance methods.
