---
layout: post
title: "Hello world, Ctypes"
description: "Using ctypes with simple data types is easy but it gets weird and complicated for other types.
 I need a hello world."
tags: python C
---

Ctypes - It's a language binding library for Python that gives Python programs ability to call C programs and vice-versa.

My first impression when I started looking into the documentation of ctypes was, 
"Oh, this looks easy. All you have to do is load the library and then start calling functions in your C program." 

Well, that was only when I was reading sample programs where they only deal with simple data types like strings and integers.
But when I began using it at my work were I needed to call C libraries from Python, I immediately got stumped when I saw
the data being passed and returned from the C functions were structs, arrays, pointers, pointers to pointers, ... 
Ah, suddenly it got complicated.

## How to retrieve a struct data from a C function

Passing integers and strings to and from a C program is straightforward.
But when it comes to struct, there is a bit of setup involved. 
Below is an example on how to retrieve a struct data from C. 
I made the example as simple as possible so that it can be better understood. 
No extra and fancy features are involved. 
I thought those things only get in the way of understanding quickly and not helping at all.
In the first example, I deliberately omitted the process of how to pass a struct so that it won't add more confusion. We'll have that in a separate example.

#### *pass_struct.c*
```
#include <stdio.h>
#include <stdlib.h>

struct data;
struct data get_data();
void free_data(struct data);

struct data {
    int status;
    char *message;
};

struct data get_data()
{
    struct data d;

    // allocate memory since we are taking it out of the function
    d.message = (char *) malloc(50 * sizeof(char));

    // fill d with information
    snprintf(d.message, 50, "%s", "hello from C");
    d.status = 5;

    return d;
}

void free_data(struct data d)
{
    printf("Freeing this string from memory: \"%s\"\n", d.message);
    free(d.message);

    printf("I'm free!\n");
}
```

Here's a C program that has a function called *get_data()*. It returns a struct containing two members: a string and an integer.
Ignore the function *free_data()* for now.

To make this callable from Python, we need to compile this program into a shared library. We'll call it *mylib.so*.

```
gcc -Wall -Werror -g -fPIC pass_struct.c -shared -o mylib.so
```


#### *get_data.py*
```python
#!/usr/bin/env python

import ctypes

class Data(ctypes.Structure):
    _fields_ = [('status', ctypes.c_int),
                ('message', ctypes.c_char_p),]

if __name__ == '__main__':
    # load the C shared library
    lib = ctypes.cdll.LoadLibrary('./mylib.so')

    lib.get_data.argtypes = None
    lib.get_data.restype = Data

    d = lib.get_data()
    print d.status, d.message
```

Here's our Python program, *get_data.py*, calling the C function *get_data()*.

To receive a struct returned by a C function, you'll need to define a Python class that inherits *ctypes.Structure*.
You'll need to set the *_fields_* attributes of that class matching the members of the struct.

The sample above has an integer and a string
as members of the struct. Before calling the function, you'll need to set the *argtypes* and *restype* attributes of the library instance. These attributes correspond to the data types of the input and the return of the C function.
Since we are not passing any data to the function, the input is set to None. And for the return,
it will be *Data* which is the name of the class we defined above.

```
lib.get_data.argtypes = None
lib.get_data.restype = Data
```

Running this Python program will give you this output:
```
$ python get_data.py

5 hello from C
```

Easy, right? Like a hello world program.

But you might say, "There is a memory leak!" 
That's true. We will take care of that when we show how to pass a struct to a C program.

### How to pass a struct data to a C function

To pass a struct to a C function, you'll need to set the *argtypes* of the name of the class Structure,
which is in our case *Data*.
It will be enclosed in square brackets since function arguments are like lists in Python.

```
lib.free_data.argtypes = [Data]
```

In the *pass_struct.c* program above, there is a function called *free_data()* that accepts a struct.
We will call this function from Python to pass a struct.

When we called the *get_data()* function, it allocated a memory to the d.message string.
When we're done using it, we need to free its memory usage or else it will be memory leak.
This is where the *free_data()* funtion comes in when it receives the struct data d.

Here's a modified version of the *get_data.py* calling the *free_data()* from Python.

#### get_data.py
```
#!/usr/bin/env python

import ctypes

class Data(ctypes.Structure):
    _fields_ = [('status', ctypes.c_int),
                ('message', ctypes.c_char_p),]

if __name__ == '__main__':
    lib = ctypes.cdll.LoadLibrary('./mylib.so')

    lib.get_data.argtypes = None
    lib.get_data.restype = Data

    d = lib.get_data()
    print d.status, d.message

    # free the memory allocated to struct d
    lib.free_data.argtypes = [Data]
    lib.free_data.restype = None
    lib.free_data(d);
```

Running this program will give you this output:
```
$ python get_data.py

5 hello from C
Freeing this string from memory: "hello from C"
I'm free!
```

I hope that was easy to follow. Put a comment below if you have any questions.

---
### Miscellaneous
When creating a class to represent a struct in C, it's nice to add the *`__repr__()`* method. 
This method will allow you to query the members of the class directly.

```
class Data(ctypes.Structure):
    _fields_ = [('status', ctypes.c_int),
                ('message', ctypes.c_char_p),]

    def __repr__(self):
        return '({0}, {1})'.format(self.status, self.message)
```

In our case, if you print the variable d, you'll get this output:
```
(5, hello from C)
```


