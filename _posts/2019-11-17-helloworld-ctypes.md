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

## How to retrieve a struct from a C function

Passing integers and strings to and from a C program is straightforward.
But when it comes to struct, there is a bit of setup involved. 
Below is an example on how to retrieve a struct data from C. 
I made the example as simple as possible so that it can be better understood. 
No fancy and extra features are involved. Only things related to the topic are included.
I thought those extra fancy things only get in the way of understanding quickly. They are not helping at all.
In the first example, I deliberately omitted the process of how to pass a struct so that it won't add more confusion. We'll have that in a separate example.

#### *pass_struct.c*
```
#include <stdio.h>
#include <stdlib.h>

struct data;
void free_data(struct data *);

struct data *get_data();

struct data {
    int status;
    char *message;
};

struct data *get_data()
{
    struct data *dp;

    // allocate memory to out struct data
    dp = malloc(sizeof(struct data));
    dp->message  = malloc(50 * sizeof(char));

    // put contents to our struct data
    snprintf(dp->message, 50, "%s", "hello from C");
    dp->status = 5;

    // return pointer to our struct data
    return dp;
}

void free_data(struct data *d)
{
    printf("Freeing members of struct from memory: %d, %s\n", d->status, d->message);
    free(d->message);
    free(d);

    printf("I'm free!\n");
}
```

Here's a C program that has a function called *get_data()*. 
It creates a struct and puts data into it. It then returns a pointer to the struct Data.
Whoever will call this function, it will have access to the struct data produced by that funtion.

Ignore the function *free_data()* for now. That will be used in the next example.

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
    lib = ctypes.cdll.LoadLibrary('./mylib.so')

    lib.get_data.argtypes = None
    lib.get_data.restype = ctypes.POINTER(Data)

    # retrieve pointer to struct
    dp = lib.get_data()

    # use contents attribute to access real data
    d = dp.contents
    print d.status, d.message
```

Here's our Python program, *get_data.py*. 

First, we need to define a Python class matching the struct in our C program.
This is done by inheriting the *ctypes.Structure* class and defining the *_fields_* attributes, where
the *_fields_* defines the members of the C struct.

When this Python program runs, the first thing it needs to do is call the method *ctypes.cdll.LoadLibrary('./mylib.so')*
to load our C shared library.

Before we call the C function, we need to set the input and return attributes of the method object.

```
lib.get_data.argtypes = None
lib.get_data.restype = ctypes.POINTER(Data)
```

The C function does not have arguments so this is set to *None*. 
The return type of the function is
a pointer to a struct. We set this as *ctypes.POINTER(Data)*.

Since the C function returns a pointer, the actual contents of our struct is available
in the *contents* attribute of the object.

```
dp = lib.get_data()
d = dp.contents
```

Finally, we can access the contents of the struct data.

```
print d.status, d.message
```

Below is a sample run of the Python program:

```
$ python get_data.py

5 hello from C
```

Easy, right? Like a hello world program.

If we are going to keep the Python program running but it no longer needs the struct data, 
that memory will still be around since that memory was allocated in our C program.
We can free up this memory by calling the *free_data()* function in our C program.

Here we can show how to pass a struct to a C program.

## How to pass a struct to a C function

For this example, instead of passing a struct, we will pass a pointer to a struct.

Like above, before calling the funtion, we need to define the input and return attributes of the method object.

```
lib.free_data.argtypes = [ctypes.POINTER(Data)]
lib.free_data.restype = None
```

Here's the update *get_data.py* program adding a call to the *free_data()* function.

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
    lib.get_data.restype = ctypes.POINTER(Data)

    # retrieve pointer to struct
    dp = lib.get_data()

    # use contents attribute to access real data
    d = dp.contents
    print d.status, d.message

    # free the memory allocated to struct d
    lib.free_data.argtypes = [ctypes.POINTER(Data)]
    lib.free_data.restype = None
    lib.free_data(dp);
```

Below is a sample run of the Python program:

```
$ python get_data.py

5 hello from C
Freeing members of struct from memory: 5, hello from C
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


