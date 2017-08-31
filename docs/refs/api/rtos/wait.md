#### Wait

Wait functions provide simple wait capabilities. The OS scheduler puts the current thread in `waiting state`, allowing another thread to execute. Even better: If there are no other threads in `ready state`, it can put the whole microcontroller to `sleep`, saving energy.

```
/** Waits for a number of seconds, with microsecond resolution (within
 *  the accuracy of single precision floating point).
 *
 *  @param s number of seconds to wait
 */
void wait(float s);

/** Waits a number of milliseconds.
 *
 *  @param ms the whole number of milliseconds to wait
 */
void wait_ms(int ms);

/** Waits a number of microseconds.
 *
 *  @param us the whole number of microseconds to wait
 */
void wait_us(int us);
```

##### Example

[![View code](https://www.mbed.com/embed/?url=https://developer.mbed.org/teams/mbed_example/code/wait_ex_1/)](https://developer.mbed.org/teams/mbed_example/code/wait_ex_1/file/7d249aa3d880/main.cpp)

##### Avoiding OS delay

When you call `wait`, your board's CPU is in a loop waiting for the required time to pass. Using the [Arm Mbed RTOS](rtos.md), you can make a call to `Thread::wait` instead.
