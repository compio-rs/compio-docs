# A story of `compio`

Long, long time ago, there was a group "Rust 众".
People there were interested in asynchronous programming, and the newest, hottest technology io-uring.
They wondered if the most popular async runtime `tokio` could use this technology as the backend.

The `tokio` community had some trials, but the interface of `tokio-uring` was different from the original `tokio` one, for safety reasons.
Traditionally, `tokio` is based on reactor model,
while io-uring is a preactor.
Preactor based IO methods require the ownership of the buffers.
Therefore, unless `tokio` finally decided to release 2.0 for the breaking change, it would never use io-uring as first class.

Wait - IOCP on Windows is also a preactor, right?
Yes, but no one really cares about that.
Windows IOCP is neither newest nor hottest.
It has been in the system kernel for 20 years.
Before io-uring, it was the only preactor among the popular operating systems.
Therefore, `tokio` decided to use the *undocumented* AFD driver, which is a satisfying reactor.

Now io-uring is here.
Unexpectedly, the greatest open source operating system chooses preactor.
It's not `tokio`'s fault, because it should keep its API stable.
Let's try to find some other async runtimes.
Unfortunately, there was no other runtime, that used io-uring correctly, and was cross-platform.

Then I came up with an idea: if there is no such runtime, why cannot I author one?
I started with `tokio-iocp` crate (don't use it please).
It was a failure, because `tokio` refused to expose `mio` interfaces, and I ended up with a runtime that consumed CPU 100% even there were no tasks to operate.
However, with the experience and lessons from the failure, I believed that I had the ability to author a new one.

I started with a runtime based on IOCP.
It should contain a struct `File` to read and write the files.
Then it was ported to Linux based on io-uring.
The API of IOCP and io-uring differs, but their natures are the same.
The API of the new runtime imitated `tokio` and `monoio`.
When the first example ran correctly on both Windows and Linux, I was very excited.
I told about the new runtime in the group.
Since then, no one in the group ever dreamed of `tokio` 2.0.

I gave this new runtime the name "compio", which means completion-based IO.
It should pronounce like /'kɔmpaio/ or /'kɔmpio/.
The name follows the non-existent convention that an async runtime should be named with a suffix "io".
Later, I also designed a logo for it.
It is a letter "C" with a plug inside, which means that it is a "socket".

By the project developed, more people showed interest in it.
Thanks to @George-Miao (Pop)'s so many efforts on `compio`.
He is a better developer and reviewer than me.
When I introduced the runtime in the group, he noticed that and decided to port it to Mac.
It was not an easy job, because there is no preactor on Mac.
He had to simulate a preactor with a reactor.
After his success, the structure of `compio-driver` finally established.

`compio` had few users at first.
It only provided mid-level interfaces, e.g., TCP streams.
If there were an HTTP client based on `compio`, it would be easier for newcomers to try `compio`.
Thought of that, I started to author `compio-http`, and finally it became `cyper`.
`cyper` is based on `compio` and `hyper`, providing HTTP client in `cyper` crate, and `axum` adapters in `cyper-axum`.

Before April Fools of 2024, I decided to make some big news.
I found that the natures of an async runtime and a GUI framework are the same.
Therefore, it should be possible to write a GUI async runtime, in which IO operations in the same thread wouldn't freeze the GUI.
I started with another new runtime, copying code from `compio`, and added some other stuffs to handle the message queue on Windows.
It worked like a charm.
Then I exposed some low-level API from `compio-runtime` to make the GUI runtime use `compio` directly.
The crate was finally ported to GTK & Qt on Linux, and Cocoa on Mac.
I called it `winio`, because it was originally a Windows-IO runtime.
`winio` *was* an April Fools' joke, but now it's a serious project, a cross-platform native GUI async framework.

The compio project is my largest project up to now.
I have spent a lot on it, dreaming it becoming an alternative to `tokio`.
It's OK if the dream doesn't come true but, at least, I hope `compio` to be a good experimental project for everyone who is interested in exploring new programming paradigms.

Thank you all the maintainers and supporters.
The project wouldn't be here without your help.

----

Berrysoft, Oct. 2024
