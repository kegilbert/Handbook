

Before we get started, this is a rather large document that delves into a lot of topics. Use the following links to jump to sections that are focused on more specifics tasks:

Table of Contents
=================

* [Why Callback](#callbacks)
* [The importance of state (a.k.a why not function pointers?)](#the-importance-of-state-aka-why-not-function-pointers)
* [The importance of explicit memory allocation and C++11 (aka Why not std::function)](#the-importance-of-explicit-memory-allocation-and-c11-aka-why-not-stdfunction)
* [Context](#context)
* [Mbed Callbacks](#mbed-callbacks)
* [Examples](#examples)
* [Ticker attach](#ticker-attach)
* [Static function](#static-function)
* [Member function](#member-function)
* [Combinatorial explosion damage control (aka why a separate class for Callbacks)](#combinatorial-explosion-damage-control-aka-why-a-separate-class-for-callbacks)
* [Callback argument passing](#callback-argument-passing)
* [API](#api)
* [Thread example with callbacks](#thread-example-with-callbacks)
* [Warnings and notes](#warnings-and-notes)

[What are callbacks?]
The importance of state (aka why no function pointers)
Combinatorial explosion damage control (aka why a separate class for Callbacks)
Why not std::function?
[Mbed Callback API]
[Mbed Callback Examples]
[Callbacks dos and don’ts]

### Callbacks

What are callbacks?

A callback is a user provided piece of code that is passed to another function. The callback allows an API to execute the user’s code in its own context. Take the following simple example.

```c++
// Create a serial object
Serial serial(USBTX, USBRX);                                                       
                                     
// A function that echoes any received data back          
 void echo() {
    while (serial.readable()) {
         serial.putc(serial.getc());                        
    }
 }                                                         
                                                           
 int main(void) {   
     // Call our function echo whenever the serial line receives data                                   
     serial.attach(&echo, Serial::RxIrq);                               
 }                                                         
```

#### The importance of state (a.k.a why not function pointers?)

Callbacks have two important pieces of information, the code to execute, and the state associated with the callback.

A common API design mistake is to use a callback type that doesn’t allow a user to attach state to a callback. The most common example of this is a simple C function pointer:


```c++
class ADC {
public:
    // Here, the adc_callback_t type is a function that takes in data
    typedef void (*adc_callback_t)(float data);


    // In this example, the ADC read function calls the user-provided callback
    // when data is available.
    void attach(adc_callback_t cb);
};
```

This API is sufficient for simple applications, but falls apart when there are multiple ADC modules available. This problem becomes especially noticeable when a user tries to reuse the same procedure for multiple callbacks.

For example, consider applying a low-pass filter to two different ADC modules

``` c++
// Here is a small running-average low-pass filter. 
float low_pass_result;
void low_pass_step(float data) {
     low_pass_result = low_pass_result*0.99 + data*0.01;
}

// Our two adc modules
ADC adc1;
ADC adc2;

// Here we can register the low-pass filter on both ADC modules
int main() {
    // Register one low pass filter
    adc1.attach(low_pass_step);

    // Register a second low pass filter
    // Problem! Now both low pass filters are sharing the same state!
    adc2.attach(low_pass_step);
}
```

Without state, callbacks offer very limited composability. In C, we can fix this by adding an additional “state” argument to the function pointer, and allow the user to pass in the opaque “state” when they register the callback.

Here’s the low-pass example using an additional argument for state.

```c++
class ADC {
   public: 
   // Here, the adc_callback_t type is a function that takes in data, as well as a pointer for state
    template <typename T>
    typedef void (*adc_callback_t)(T *state, float data);


    // In this example, the ADC read function calls the user-provided callback
    // when data is available.
    template <typename T>
    void attach(adc_callback_t<T> cb, T *state); 
};

// Here is a small running-average low-pass filter. 
void low_pass_step(float *result, float data) {
     *result = *result*0.99 + data*0.01;
}

// Our two adc modules
ADC adc1;
ADC adc2;

// Our two low-pass filter results
float low_pass_result1;
float low_pass_result2;

// Here we can register the low-pass filter on both ADC modules
Int main() {
    // Register one low pass filter
    adc1.attach(low_pass_step, &low_pass_result1);

    // Register a second low pass filter, no more issues!
    adc2.attach(low_pass_step, &low_pass_result1);
}
```

One of the core features of C++ is the encapsulation of this “state” in classes, with operations that modify the state being represented as member functions in the class. Unfortunately, member function pointers are not compatible with standard function pointers, but we can rewrite the low-pass example to use member function pointers, allowing users to pass in state as a C++ object.

Here’s the low-pass example rewritten to use member function pointers.

```c++
class ADC {
public:
    // Here, the adc_callback_t type is a function that takes in data
    template <typename T>
    typedef void (T::*adc_callback_t)(float data);


    // In this example, the ADC read function calls the user-provided callback
    // when data is available.
    template <typename T>
    void attach(T *obj, adc_callback_t<T> cb);
};

class LowPass {
   float result;

public:
    // Move the low pass filter implementation to the ADC module
    void step(float data) {
        result = result*0.99 + data*0.01;
    }
};


// Our two adc modules
ADC adc1;
ADC adc2;

// Our two low-pass filters
LowPass low_pass1;
LowPass low_pass2;

// Here we can register the low-pass filter on both ADC modules
int main() {
    // Register one low pass filter
    adc1.attach(&low_pass1, &LowPass::step);
    adc2.attach(&low_pass2, &LowPass::step);
}
```

#### Combinatorial explosion damage control (aka why a separate class for Callbacks) 

While heralded for its support of high-level design patterns, C++ makes it surprisingly difficult to support all of the standard function types.

1. State is important, so need to support either C-style function pointers with state, or C++ member function pointers.

2. Stateless callbacks are just as common, but passing a stateless callback as a member function function requires writing a lot of boilerplate code and instantiating an empty class. So we need to also support a standard function pointer.

3. Another design pattern you may see is the function object, a class that overrides the function call operator. We can expect the user to pass function objects as C++ member function pointers if needed.

4. A useful C++ feature is the enforcement of const-correctness, but this becomes unfortunately complicated with the state associated with callbacks. A C++ API needs to support both the const and non-const versions of member function pointers.

5. Another C++ feature is volatile-correctness in case the underlying state must be volatile, but if necessary we can probably expect the user to hide volatile members inside of a non-volatile class.

Long story short, C++ requires a large set of overloads to support all of the standard function types. It is very error-prone and just flat out unreasonable to expect a new library author to add all of these overloads to every function that could take in a callback.

Fortunately, C++ does provide the tools to delegate this problem to a single class. This class is the Callback class. The Callback class should be familiar to users of the std::function class that was introduced in C++11, but is available for older versions of C++.

An overly-simplified description of the Callback class is that is contains all of this madness so you don’t have to.

Here’s the low-pass filter example rewritten to use the callback class:

``` c++
class ADC {
public:
    // In this example, the ADC read function calls the user-provided callback
    // when data is available.
    void attach(Callback<void(float)> cb);
};

class LowPass {
   float result;

public:
    // Move the low pass filter implementation to the ADC module
    void step(float data) {
        result = result*0.99 + data*0.01;
    }
};


// Our two adc modules
ADC adc1;
ADC adc2;

// Our two low-pass filters
LowPass low_pass1;
LowPass low_pass2;

// Here we can register the low-pass filter on both ADC modules
int main() {
    // Register one low pass filter
    adc1.attach(callback(&low_pass1, &LowPass::step));
    adc2.attach(callback(&low_pass2, &LowPass::step));
}
```

### How to create callbacks

First we need to understand the syntax of the Callback type. The Callback type is a templated type parameterized by a C++ function declaration:

``` c++
// Callback</*return type*/(/*parameters*/)> cb;
Callback<int(float)> cb;      	   // A callback that takes in a float and returns an int
Callback<void(float)> cb;     	   // A callback that takes in a float and returns nothing
Callback<int()> cb;           	   // A callback that takes in nothing and returns an int
Callback<void(float, float)> cb;  // A callback that takes in two floats and returns nothing
```

You can create a Callback directly from a C function or function pointer with the same type:

``` c++
void dosomething(int) {
    // do something
}

Callback<void(int)> cb(dosomething);
```

If an API provides a function that takes in a callback, we can just pass in a C function or function pointer with the same type:

``` c++
class ADC {
    void attach(Callback<void(float)> cb);
};

void dosomething(float) {
    // do something
}

ADC adc;
adc.attach(dosomething);
```

But what about state? The Callback type also supports passing a state pointer for a function. This state can be either a pointer to an object that is passed to a member function, or it can be a pointer passed to a C-style function.

Since this form of creating Callbacks requires two arguments, the Callback will need to be created explicitly using the Callback constructor. The Callback also comes with the lowercase callback function, which creates callbacks based on the arguments type, and avoids the need to repeat the template type.

We can create a callback with a member function

``` c++
class Thing {
    int state;
    void catinthehat(float f) {
        state = // do something
    }
}

// We can create a Callback with the Callback constructor
Thing thing1;
adc.attach(Callback<void(int)>(&thing1, &Thing::catinthehat));

// Or we can create a Callback with the callback function to avoid repeating ourselves
Thing thing2;
adc.attach(callback(&thing2, &Thing::catinthehat));
```

Or we can just pass the state to a C-style function

``` c++
struct thing_t {
    int state;
}

void catinthehat(thing_t *thing, float f) {
    thing->state = // do something
}

// We can create a Callback with the Callback constructor
thing_t thing1;
adc.attach(Callback<void(int)>(catinthehat, &thing1));

// Or we can create a Callback with the callback function to avoid repeating ourselves
thing_t thing2;
adc.attach(callback(catinthehat, &thing2));
```

An important thing to note, is that this state is restricted to a single pointer. This means you can’t bind both an object and argument to a callback.

``` c++
 // Does not work
adc.attach(callback(&thing, &Thing::dosomething, &arg));
```
If you need to pass multiple arguments to a callback, and you can’t store the arguments in the class, you can create a struct that contains all of the arguments and pass a pointer to that. However, you will need to handle the memory allocation yourself.

``` c++
// Create a struct that contains all of the state needed for “dosomething”
struct dosomething_arguments {
    Thing *thing;
    int arg1;
    int arg2;
};

// Create a function that calls “dosomething” with the arguments
void dosomething_with_arguments(struct dosomething_arguments *args) {
    args->thing->dosomething(args->arg1, args->arg2);
}


// Allocate arguments and pass to callback
struct dosomething_arguments args = { &thing, arg1, arg2 };
adc.attach(callback(dosomething_with_arguments, &args)); // yes
```

### How to call callbacks

Callbacks overload the function call operator, so you can just call a Callback like you would a normal function:

```c++
void callme(Callback<void(float)> cb) {
    cb(1.0f);
}
```

The only thing to really watch out for is that the Callback type has a null Callback, just like a null function pointer. Uninitialized callbacks are considered null and will assert if called. If you want a call to always succeed you will need to check if it is null first.

``` c++
void callmemaybe(Callback<void(float)> cb) {
    If (cb) {
        cb(1.0f);
    }
}
```

The Callback class is what’s known in C++ as a “Concrete Type”, that is, the Callback class is lightweight enough to be passed around like an int, pointer, or other primitive type.

```c++
class Thing {
private:
    Callback<void(int)> _cb;

public:
    void attach(Callback<void(int)> cb) {
         _cb = cb
    }

    void dothething(int arg) {
        If (_cb) {
            _cb(arg);
        }
    }
}
```

### SONAR EXAMPLE

```c++
#include <mbed.h>

class Sonar {
    DigitalOut *ping;
    InterruptIn  *echo;
    Timer      timer;
    Ticker     delay;
    Ticker     call;
    bool       ping_done;
    bool       ready;
    int32_t    begin;
    int32_t    end;
    int32_t    distance;

public:
    Sonar(PinName ping_pin, PinName echo_pin) {
        ping = new DigitalOut(ping_pin);
        echo = new InterruptIn(echo_pin);
        ping_done = false;
        ready     = false;
        ping->write(0);
        echo->rise(callback(this, &Sonar::echo_in));
        echo->fall(callback(this, &Sonar::echo_fall));
        distance = -1;
    }
   
    void start(void) {
        call.attach(callback(this, &Sonar::background_read), 0.01f);
    }
    
    void stop(void) {
        call.detach();
    }
 
    void ping_toggle(void) {
        ping->write(0);
        ping_done = true;
        delay.detach();
    }
    
    void echo_in(void) {
        timer.reset();
        timer.start();
        begin = timer.read_us();
    }

    void echo_fall(void) {
        end = timer.read_us();
        timer.stop();
        ready = true;
    }

    void background_read(void) {
        ping->write(1);
        delay.attach(callback(this, &Sonar::ping_toggle), 10.0e-6);
        ping_done = false;
        while(!ready) {}
        distance = end - begin;
    }

    int32_t read(void) {
        return distance / 58;
    }
};



int main() {
    Sonar sonar(D5, D6);
    sonar.start();

    /* Do some stuff */   

    float distance = sonar.read();
    sonar.stop();
}

``` 

### API

[![View code](https://www.mbed.com/embed/?type=library)](https://docs.mbed.com/docs/mbed-os-api/en/mbed-os-5.5/api/classmbed_1_1Callback_3_01R_07A0_08_4.html)

###### Thread example with callbacks

The Callback API provides a convenient way to pass arguments to spawned threads.  

[![View code](https://www.mbed.com/embed/?url=https://developer.mbed.org/teams/mbed_example/code/rtos_threading_with_callback/)](https://developer.mbed.org/teams/mbed_example/code/rtos_threading_with_callback/file/d4b2a035ffe3/main.cpp)

###### Serial passthrough example with callbacks
[![View code](https://www.mbed.com/embed/?url=https://developer.mbed.org/teams/mbed_example/code/rtos_threading_with_callback/)](https://developer.mbed.org/users/mbedAustin/code/SerialPassthrough/file/96cb82af9996/main.cpp)


