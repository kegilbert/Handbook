#### Mutex

A [`Mutex`](https://docs.mbed.com/docs/mbed-os-api/en/mbed-os-5.5/api/classrtos_1_1Mutex.html) is used to synchronize the execution of threads, for example to protect the access to a shared resource.

<span class="warnings"> **Warning - ISR:** The `Mutex` methods cannot be called from interrupt service routines (ISR). In the current version of Mbed OS, if you attempt to use a mutex from within an ISR, nothing happens; attempts to lock a mutex succeed immediately, regardless of whether the lock is actually free. In other words, if you acquire a mutex lock in an ISR, you can break the thread safety mechanisms and introduce race-conditions into an otherwise safe piece of code. Future versions of Mbed OS will provide warnings and ultimately prevent this from happening. </span>

<span class="images">![](Images/Thread/Mutex.png)</span>

##### Mutex example

Use Mutex to protect printf().

[![View code](https://www.mbed.com/embed/?url=https://developer.mbed.org/teams/mbed_example/code/rtos_mutex/)](https://developer.mbed.org/teams/mbed_example/code/rtos_mutex/file/1ae0d86d2020/main.cpp)

<span class="notes">**Note:** C standard library Mutexes</br>The Arm C standard library already has Mutexes in place to protect the access to `stdio`, so on the LPC1768 the above example is not necessary. On the other hand, the LPC11U24 does not provide default `stdio` Mutexes, making the above example a necessity. </span>

<span class="warnings">**Warning:** `stdio`, `malloc` and `new` in ISR</br>Because of the mutexes in the Arm C standard library, you cannot use `stdio` (`printf`, `putc`, `getc` and so on), `malloc` and `new` in ISR. </span>

##### Mutex class reference

[![View code](https://www.mbed.com/embed/?type=library)](https://docs.mbed.com/docs/mbed-os-api/en/mbed-os-5.5/api/classrtos_1_1Mutex.html)
