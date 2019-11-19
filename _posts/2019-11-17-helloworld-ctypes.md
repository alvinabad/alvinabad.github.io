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

The following are examples of how to pass a struct data from C to Python and vice-versa. 
I made the examples as simple as possible so that they can be better understood. 
No fancy and extra features are involved. Only things related to the topic are in the code.
I thought fancy examples only add to the confusion and gets in the way of understanding the topic.

I would assume that you know both C and Python. I will not explain how the program works on each side.
I will only explain the things how the hooks are set up to link each other.

#### *pass_struct.c*
```
#include <stdio.h>
#include <stdlib.h>

struct data;

struct data *get_data();
void send_data(struct data *);
void free_data(struct data *);

struct data {
    int status;
    char *message;
};

struct data *get_data()
{
    struct data *d;

    // allocate memory to out struct data
    d = malloc(sizeof(struct data));
    d->message  = malloc(50 * sizeof(char));

    // put contents to our struct data
    snprintf(d->message, 50, "%s", "hello from C");
    d->status = 5;

    // return pointer to our struct data
    return d;
}

void send_data(struct data *d)
{
    printf("Received data in C: %d, %s\n", d->status, d->message);
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

Ignore the other functions for now. They will be used in the next examples.

To make the function *get_data()* callable from Python, we need to compile this program into a shared library. 
We'll call it *mylib.so*.

```
gcc -Wall -Werror -g -fPIC pass_struct.c -shared -o mylib.so
```


#### *pass_struct.py*
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

Here's a Python program, named *pass_struct.py*. 

First, we need to define a Python class matching the struct in our C program.
This is done by inheriting the *ctypes.Structure* class and defining the *_fields_* attributes, where
the *_fields_* defines the members of the C struct.

To access the C program in a share library, we need to call this method:

```
lib = ctypes.cdll.LoadLibrary('./mylib.so')
```

The lib instance created will give the Python program access to all the functions in the loaded library.
Calling the functions will be as easy as running it like this:

```
lib.get_data()
```

But before we call the C function, we need to set the argtypes and restype attributes of the method object.
These corresponds to the input arguments and return of the C function, respectively.
For the *get_data()_* function, we set it like this:

```
lib.get_data.argtypes = None
lib.get_data.restype = ctypes.POINTER(Data)
```

The C function does not have arguments so it is set to *None*. 
The return type of the function is
a pointer to a struct. We set this as *ctypes.POINTER(Data)*.

Since the C function returns a pointer, the actual contents of our struct will be available
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
$ python pass_struct.py

5 hello from C
```

Easy, right? Like a hello world program.

Next, let's show how to pass a struct.

## How to pass a struct to a C function

In our C program, we have function named *send_data()*. 
This function accepts a pointer to a struct.

Similarly, we need to set the argtypes and restype attributes of the function.

```
lib.free_data.argtypes = [ctypes.POINTER(Data)]
lib.free_data.restype = None
```

Here's an updated *pass_struct.py* program with a call to the *send_data()* function.

#### pass_struct.py
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

    # create new Data
    new_d = Data()
    new_d.message = "hello, alvin"
    new_d.status = 100

    # define attributes of send_data method
    lib.send_data.argtypes = [ctypes.POINTER(Data)]
    lib.send_data.restype = None

    # create pointer to new_d
    new_dp = ctypes.pointer(new_d)

    # send new_d
    lib.send_data(new_dp)
```

Here we created a new Data, named *new_d*.

Since the function accepts a pointer, we need to convert *new_d* into a pointer.
This is done by calling the ctypes method, *ctypes.pointer()*. 
This pointer is then supplied to the *send_data()* function.

```
# create pointer to new_d
new_dp = ctypes.pointer(new_d)

# send new_d
lib.send_data(new_dp)
```

Below is a sample run of the Python program:

```
$ python get_data.py

5 hello from C
Received data in C: 100, hello, alvin
```

Easy, right? I hope that was easy to follow. Put a comment below if you have any questions.

---
## Miscellaneous
### Memory Management
You'll notice that there is function named *free_data()*.
This function can free up the memory allocated to a struct data.

Since we allocated heap memory in *get_data()*, we can use this to free up that memory,
and call it from Python:

```
# free the memory allocated to struct d
lib.free_data.argtypes = [ctypes.POINTER(Data)]
lib.free_data.restype = None
lib.free_data(dp)
```

### Other attributes of ctypes.Structure
When creating a class to represent a struct in C, it's nice to add the *`__repr__()`* method. 
This method will allow you to query the members of the class directly.

```
class Data(ctypes.Structure):
    _fields_ = [('status', ctypes.c_int),
                ('message', ctypes.c_char_p),]

    def __repr__(self):
        return '({0}, {1})'.format(self.status, self.message)
```

Using our example, if you print the variable d, you'll get this output:
```
(5, hello from C)
```


