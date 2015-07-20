

    #### FUNCTIONAL CONCEPTS FOR PYTHON
    # Taken from:
    # Functional  Programming  in Python by David Mertz - O'Reilly


    ### ENCAPSULATION
    
    # configure the data to start with 
    collection = get_initial_state() 
    state_var = None 
    for datum in data_set:    
        if condition(state_var):        
            state_var = calculate_from(datum)        
            new = modify(datum, state_var)        
            collection.add_to(new)    
        else:        
            new = modify_differently(datum)
            collection.add_to(new)
    # Now actually work with the data 
    for thing in collection:    
        process(thing) 
        
    #We might simply remove the “how” of the data construction from the current scope, 
    #and tuck it away in a function that we can think about in isolation 
    #(or not think about at all once it is sufficiently abstracted). 
    #For example:
    # tuck away construction of data
    def make_collection(data_set):    
        collection = get_initial_state()    
        state_var = None    
        for datum in data_set:
            if condition(state_var):
                state_var = calculate_from(datum, state_var)
                new = modify(datum, state_var)
                collection.add_to(new)
            else:            
                new = modify_differently(datum)
                collection.add_to(new)    
        return collection 
    
    # Now actually work with the data 
    for thing in make_collection(data_set):    
        process(thing) 


    ### COMPREHENSIONS
    
    #if our original code was:
    collection = list() 
    for datum in data_set:    
        if condition(datum):        
            collection.append(datum)    
        else:        
            new = modify(datum)        
            collection.append(new) 
            
    #Somewhat more compactly we could write this as:
    collection = [d if condition(d) else modify(d) for d in data_set] 


    ### GENERATORS
    
    # they are also lazy. That is to say that they are merely a description of “how to get the data” that is 
    # not realized until one explicitly asks for it, either by calling .next() 
    
    log_lines = (line for line in read_line(huge_log_file) if complex_condition(line))
    
    # Obviously, this generator comprehension also has imperative versions, for example:
    def get_log_lines(log_file):
        line = read_line(log_file)    
        while True:         #  the behind-the-scenes “how” of a while loop over an iteratable (generator above)
            try:            
                if complex_condition(line):                
                    yield line   
                line = read_line(log_file)        
            except StopIteration:            
                raise
                
    log_lines = get_log_lines(huge_log_file) 
    
    # We could do this with a class that had .__next__() and .__iter__() methods. For example:
    class GetLogLines(object):    
        def __init__(self, log_file):        
            self.log_file = log_file        
            self.line = None    
        def __iter__(self):        
            return self    
        def __next__(self):        
            if self.line is None:
                self.line = read_line(log_file)
            while not complex_condition(self.line):
                self.line = read_line(self.log_file)
            return self.line
            
    log_lines = GetLogLines(huge_log_file)
    


    ### DICTS AND SETS
    
    # dictionaries and sets can be created “all at once” rather than by repeatedly calling .update() or .add() in a loop. For example:
    {i:chr(65+i) for i in range(6)}


    {chr(65+i) for i in range(6)}


    ### RECURSION
    
    # Python lacks an internal feature called tail call elimination that makes deep recursion computationally efficient 
    # in some languages. Let us find a trivial example where recursion is really just a kind of iteration:
    def running_sum(numbers, start=0):    
        if len(numbers) == 0:        
            print()        
            return    
        total = numbers[0] + start    
        print(total, end=" ")    
        running_sum(numbers[1:], total)
    
    running_sum([5,6,7])


    # A slightly less trivial example, factorial in recursive and iterative style:
    def factorialR(N):    
        "Recursive factorial function"    
        assert isinstance(N, int) and N >= 1    
        return 1 if N <= 1 else N * factorialR(N-1)
    
    print(factorialR(5))
    


    def factorialI(N):    
        "Iterative factorial function"    
        assert isinstance(N, int) and N >= 1    
        product = 1    
        while N >= 1:        
            product *= N        
            N -= 1    
        return product
    
    print(factorialI(5))
    
    # It is not a good idea in Python—most of the time—to use recursion merely for “iteration by other means.”


    ### ALTERNATIVES TO FOR LOOPs (when it is a good idea)
    
    ## using map()
    
    # If we simply call a function inside a for loop, the built-in higherorder function map() comes to our aid:
    for e in it:    # statement-based loop    
        func(e) 
        
    # The following code is entirely equivalent to the functional version, except there is no repeated rebinding of the variable e involved, and hence no state:
    map(func, it)   # map()-based "loop"


    ## A similar technique is available for a functional approach to sequential program flow:
    # let f1, f2, f3 (etc) be functions that perform actions 
    # an execution utility function 
    do_it = lambda f, *args: f(*args) # map()-based action sequence 
    map(do_it, [f1, f2, f3]) 


    # We can combine the sequencing of function calls with passing arguments from iterables:
    hello = lambda first, last: print("Hello", first, last) 
    bye = lambda first, last: print("Bye", first, last) 
    _ = list(map(do_it, [hello, bye], ['David','Jane'], ['Mertz','Doe'])) 
    
    do_all_funcs = lambda fns, *args: [list(map(fn, *args)) for fn in fns] 
    _ = do_all_funcs([hello, bye], ['David','Jane'], ['Mertz','Doe']) 


    ### CALLABLES
    
    #  Python actually gives us several different ways to create functions, or at least something very function-like (i.e., that can be called). They are: 
    # • Regular functions created with def and given a name at definition time 
    # • Anonymous functions created with lambda 
    # • Instances of classes that define a __call()__ method 
    # • Closures returned by function factories 
    # • Static methods of instances, either via the @staticmethod decorator or via the class __dict__ 
    # • Generator functions 
    
    ## Lambdas and named functions
    def hello1(name): 
        print("Hello", name) 
        
    hello2 = lambda name: print("Hello", name) 
    
    hello1('David')
    hello2('David')
    # The only real difference is in .__qualname__ attribute


    ### CLOSURES and CLASSES
    
    # Let us construct a toy example that shows this, something just past a “hello world” of the different styles:
    # A class that creates callable adder instances 
    
    class Adder(object):    
        def __init__(self, n):        
            self.n = n    
        def __call__(self, m):        
            return self.n + m 
    
    add5_i = Adder(5)       
    # "instance" or "imperative" We have constructed something callable that adds five to an argument passed in. Seems simple and mathematical enough. Let us also try it as a closure


    def make_adder(n):    
        def adder(m):        
            return m + n    
        return adder 
    
    add5_f = make_adder(5)  
    # "functional" So far these seem to amount to pretty much the same thing, but the mutable state in the instance provides a attractive nuisance:
    


    add5_i(10) 
    add5_f(10)          # only argument affects result 15 
    add5_i.n = 10       # state is readily changeable 
    add5_i(10)          # result is dependent on prior flow 20 
    
    # once the object exists, the closure behaves in a pure functional way, while the class instance remains 
    # state dependent


    ### METHODS OF CLASSES
    
    # accessors are callables with a limited use (from a functional programming perspective) in that they take no 
    # arguments as getters, and return no value as setters:
    class Car(object):    
        def __init__(self):        
            self._speed = 100
        @property    
        def speed(self):        
            print("Speed is", self._speed)        
            return self._speed
        @speed.setter    
        def speed(self, value):        
            print("Setting to", value)        
            self._speed = value
    
    # >> car = Car()  
    # >>> car.speed = 80  # Odd syntax to pass one argument 
    # Setting to 80 
    # >>> x = car.speed 
    # x is 80
    
    


    ### STATIC METHODS
    
    # One use of classes and their methods that is more closely aligned with a functional style of programming is to use 
    # them simply as namespaces to hold a variety of related functions:
    
    import math 
    class RightTriangle(object):    
        "Class used solely as namespace for related functions"    
        @staticmethod    
        def hypotenuse(a, b):        
            return math.sqrt(a**2 + b**2)
        @staticmethod    
        def sin(a, b):        
            return a / RightTriangle.hypotenuse(a, b)
        @staticmethod    
        def cos(a, b):        
            return b / RightTriangle.hypotenuse(a, b)


    ### GENERATOR FUNCTIONS
    
    # A special sort of function in Python is one that contains a yield statement, which turns it into a generator. 
    # What is returned from calling such a function is not a regular value, but rather an iterator that produces a 
    # sequence of values as you call the next() function on it or loop over it.
    
    def get_primes(): 
        "Simple lazy Sieve of Eratosthenes" 
        candidate = 2 
        found = [] 
        while True: 
            if all(candidate % prime != 0 for prime in found): 
                yield candidate 
                found.append(candidate) 
            candidate += 1
    
    primes = get_primes() 
    print(next(primes), next(primes), next(primes))
    # (2, 3, 5) 
    for _, prime in zip(range(10), primes): 
        print(prime, end=" ") 
    #    7 11 13 17 19 23 29 31 37 41 

    2 3 5
    7 11 13 17 19 23 29 31 37 41 


    ### MULTIPLE DISPATCHING
    
    # A very interesting approach to programming multiple paths of execution is a technique called “multiple dispatch” 
    # or sometimes “multimethods.” The idea here is to declare multiple signatures for a single function and call the 
    # actual computation that matches the types or properties of the calling arguments. 
    # Note: used to avoid many if-else statements to test matching 
    
    from multipledispatch import dispatch
    
    class Thing(object): pass 
    class Rock(Thing): pass
    class Paper(Thing): pass 
    class Scissors(Thing): pass
    
    @dispatch(Rock, Rock) 
    def beats3(x, y): 
        return None
    
    @dispatch(Rock, Paper) 
    def beats3(x, y): 
        return y
    
    @dispatch(Rock, Scissors) 
    def beats3(x, y): 
        return x
    
    @dispatch(Paper, Rock) 
    def beats3(x, y): 
        return x
    
    @dispatch(Paper, Paper) 
    def beats3(x, y): 
        return None
    
    @dispatch(Paper, Scissors) 
    def beats3(x, y): 
        return x
    
    @dispatch(Scissors, Rock) 
    def beats3(x, y): 
        return y
    
    @dispatch(Scissors, Paper) 
    def beats3(x, y): 
        return x
    
    @dispatch(Scissors, Scissors) 
    def beats3(x, y): 
        return None
    
    @dispatch(object, object) 
    def beats3(x, y):    
        if not isinstance(x, (Rock, Paper, Scissors)):        
            raise TypeError("Unknown first thing")    
        else:        
            raise TypeError("Unknown second thing")
    
    rock, paper = Rock(), Paper()        
    print(beats3(rock, paper))
    # <__main__.DuckPaper at 0x103b894a8> 
    # >>> beats3(rock, 3) 
    # TypeError: Unknown second thing
    

    <__main__.Paper object at 0x0000000003944C18>
    


    #### LAZY EVALUATION
    
    ## Custom sequence using "abc"
    # Given the get_primes() generator function discussed earlier, we might write our own container to simulate the same 
    # thing, for example:
    from collections.abc import Sequence 
    class ExpandingSequence(Sequence):    
        def __init__(self, it):        
            self.it = it        
            self._cache = []    
        def __getitem__(self, index):        
            while len(self._cache) <= index:            
                self._cache.append(next(self.it))        
            return self._cache[index]    
        def __len__(self):        
            return len(self._cache) 
    
    # This new container can be both lazy and also indexible:
    primes = ExpandingSequence(get_primes())
    for _, p in zip(range(10), primes):     
        print(p, end=" ") 
    # 2 3 5 7 11 13 17 19 23 29 
    primes[10]  # 31 
    primes[5]   # 13 
    len(primes) # 11 
    primes[100] # 547 
    len(primes) # 101


    ## Custom iterator
    
    # writing custom iterators as generator functions is most natural. However, we can also create custom classes that 
    # obey the protocol; often these will have other behaviors (i.e., methods) as well, but most such behaviors 
    # necessarily rely on statefulness and side effects to be meaningful
    
    from collections.abc import Iterable 
    class Fibonacci(Iterable):    
        def __init__(self):        
            self.a, self.b = 0, 1        
            self.total = 0    
        def __iter__(self):        
            return self    
        def __next__(self):        
            self.a, self.b = self.b, self.a + self.b        
            self.total += self.a        
            return self.a    
        def running_sum(self):        
            return self.total
        
    fib = Fibonacci() 
    # fib.running_sum() # 0 
    for _, i in zip(range(10), fib): 
        print(i, end=" ") 
    # >>> 1 1 2 3 5 8 13 21 34 55 
    # fib.running_sum() # 143 
    # next(fib) # 89 
    
    # Note:  a class that both implements the iterator protocol and also provides an additional method to return 
    # something stateful about its instance. This approach can be usefull but not functional (because statefullness).

    1 1 2 3 5 8 13 21 34 55 


    ## Module: itertools 
    # The module itertools is a collection of very powerful—and carefully designed—functions for performing iterator 
    # algebra. That is, these allow you to combine iterators in sophisticated ways without having to concretely 
    # instantiate anything more than is currently required
    # https://docs.python.org/3.5/library/itertools.html
    
    def fibonacci():     
        a, b = 1, 1      
        while True:          
            yield a          
            a, b = b, a+b
    
    print(type(fibonacci()))
    print('__iter__' in dir(fibonacci()) and '__next__' in dir(fibonacci()) )
    
    from itertools import tee, accumulate 
    s, t = tee(fibonacci()) 
    pairs = zip(t, accumulate(s)) 
    for _, (fib, total) in zip(range(7), pairs):      
        print(fib, total) 
    
    # NOTE: zip(), map(), filter(), and range() (which is, in a sense, just a terminating itertools.count()) could well 
    # live in itertools if they were not built-ins. That is, all of those functions lazily generate sequential items 
    # (mostly based on existing iterables) without creating a concrete sequence
    
    

    <class 'generator'>
    True
    1 1
    1 2
    2 4
    3 7
    5 12
    8 20
    13 33
    


    #### Higher Order Functions
    
    #  a higher-order function is simply a function that takes one or more functions as arguments and/or produces a 
    # function as a result. 
    
    # Classic "FP-style" 
    transformed = map(tranformation, iterator) 
    # Comprehension 
    transformed = (transformation(x) for x in iterator)
    # Classic "FP-style" 
    filtered = filter(predicate, iterator) 
    # Comprehension 
    filtered = (x for x in iterator if predicate(x)) 
    # The function functools.reduce() is very general, very powerful, and very subtle to use to its full power. It takes 
    # successive items of an iterable, and combines them in some way. The most common use case for reduce() is probably 
    # covered by the built-in sum(), which is a more compact spelling of:
    from functools import reduce 
    total = reduce(operator.add, it, 0) 
    # total = sum(it) 
    # It may or may not be obvious that map() and filter() are also a special cases of reduce(). That is:
    add5 = lambda n: n+5 
    reduce(lambda l, x: l+[add5(x)], range(10), []) [5, 6, 7, 8, 9, 10, 11, 12, 13, 14] 
    # simpler: map(add5, range(10)) 
    isOdd = lambda n: n%2 
    reduce(lambda l, x: l+[x] if isOdd(x) else l, range(10), []) [1, 3, 5, 7, 9] 
    # simpler: filter(isOdd, range(10))


    ## DECORATORS
    
    # probably the most common use of higher-order functions in Python is as decorators. 
    
    # Taken from: http://thecodeship.com
    
    def strong_decorate(func):
        '''first decorator'''
        def func_wrapper(name):
            return "<strong>{0}</strong>".format(func(name))
        return func_wrapper
    
    def p_decorate(func):
        '''second decorator'''
        def func_wrapper(name):
            return "<p>{0}</p>".format(func(name))
        return func_wrapper
    
    @strong_decorate
    @p_decorate
    def get_text(name):
        '''sample function'''
        return "lorem ipsum, {0} dolor sit amet".format(name)
    
    #my_get_text = p_decorate(get_text)
    print(get_text)
    print(get_text("John"))
    
    # A function that takes another function as an argument, generates a new function, augmenting the work of the original 
    # function, and returning the generated function so we can use it anywhere.

    <function strong_decorate.<locals>.func_wrapper at 0x000000000379AEA0>
    <strong><p>lorem ipsum, John dolor sit amet</p></strong>
    


    ## Using self and *args and **kargs
    
    def p_decorate(func):
       def func_wrapper(*args, **kwargs):
           return "<p>{0}</p>".format(func(*args, **kwargs))
       return func_wrapper
    
    class Person(object):
        def __init__(self):
            self.name = "John"
            self.family = "Doe"
    
        @p_decorate
        def get_fullname(self):
            return self.name+" "+self.family
    
    my_person = Person()
    
    print(my_person.get_fullname())

    <p>John Doe</p>
    


    ## Passing arguments to decorators
    
    # Looking back at the example before the one above, you can notice how redundant the decorators in the example are. 
    # 2 decorators each with the same functionality but wrapping the string with different tags. We can definitely do much 
    # better than that. Why not have a more general implementation for one that takes the tag to wrap with as a string? 
    # Yes please!
    
    #from functools import wraps
    
    def tags(tag_name):
        def tags_decorator(func):
            #@wraps(func)
            def func_wrapper(name):
                return "<{0}>{1}</{0}>".format(tag_name, func(name))
            return func_wrapper
        return tags_decorator
    
    @tags("p")
    @tags("strong")
    def get_text(name):
        return "Hello "+name
    
    print(get_text("John"))
    
    print(get_text.__name__)
    
    # NOTE: try to uncomment the import and the "wraps" decorator and see how the function name is assigned properly 
    # (good for debugging).
    
    # Links:
    # https://github.com/mjhea0/python-decorators
    # http://simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/
    # https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator
    ## A guide to develop with Decorators:
    # http://www.artima.com/weblogs/viewpost.jsp?thread=240808
    # http://www.artima.com/weblogs/viewpost.jsp?thread=240845
    # http://www.artima.com/weblogs/viewpost.jsp?thread=241209

    <p><strong>Hello John</strong></p>
    func_wrapper
    


    
