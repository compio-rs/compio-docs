# Driver
`compio-driver` provides 3 kinds of drivers, currently.
On Windows, the IOCP driver is selected.
On Linux, users can decide whether to use io-uring or epoll (powered by `polling`), or let the crate decide itself.
For other circumstances, a reactor driver powered by `polling` is used.
The driver type could be retrieved dynamically by `compio::driver::DriverType`.

Unless you want to write your own runtime based on a low level proactor, using `compio-driver` directly is not a good choice.
The examples of `compio` *does* provide an example using `compio-driver` directly, dealing with platform differences itself.

## Operations
The crate itself exposes low-level proactor API.
There have been already some wrapped operations, called `OpCode`.
The interface of operations may differ for different platforms and drivers.

Note that currently the `OpCode`s are boxed inside the drivers.
The allocation could not be avoided, because we need to erase the different types of buffers.

## Thread pools
Different drivers support different kinds of operations.
For example, the polling driver doesn't support async file IO.
Therefore, the file operations in polling driver are actually spawned into a thread pool.

There is a limitation on number of threads in the thread pool.
The limitation could be modified manually when creating the `Proactor`.
If the number of threads exceeds the limitation, the operation may fail immediately.

## Cancellation
The `Proactor` provides a simple cancellation mechanism.
The cancellation of an operation may succeed or fail.
The cancellation behavior may differ by drivers.
