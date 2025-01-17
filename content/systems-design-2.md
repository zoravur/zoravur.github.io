+++
title = 'Napkin Math 2: Applying It to Computers'
date = 2025-01-17T08:50:16-05:00
draft = false
+++

In this post, I will be explaining the technique of estimating performance from first principles. It's based heavily on the following talk:
https://www.youtube.com/watch?v=IxkSlnrRFqc. 

To recap, the purpose of napkin math is to provide a quick approximation to within an order of magnitude of the real answer, for the purposes
of quickly testing the feasibility of the design of a system. Last time, we talked about Fermi estimation, which is (according to my definition)
just napkin math where you're primarily multiplying, and isn't necessarily focus on computers.

But for our purposes, the goal has always been to eventually estimate the performance of computer systems. Take, for example, the following 
problem, also presented within the talk:

- There is a local client, written in Rust (take this to mean, no performance issues), talking to a Redis instance on the same machine.
- The Rust client is repeatedly measuring the latency of the GET operation, transferring an 8 byte payload each time.
- How long can we expect this to take?

I had memorized some base rates / did some googling, before arriving at my answer. If you don't have the base rates off the top of your head,
I encourage you to do the same. Here was my line of reasoning:

> - We may assume no performance issues with the client
> - 8 bytes is a trivially small payload, meaning we can ignore the data throughput speed here, and focus solely on latency
> - Redis is an in-memory database
> - Memory access is on the order of 100 ns.
> - How does Redis communicate with the client? Is it a separate process?
> - (Googles)
> - Redis is a separate process. Therefore it must rely on ipc.
> - What is the latency of IPC?
> - (ChatGPTs)
> - Okay, unix domain sockets are 1-10 us, while tcp/udp loobpack is 10-100 us.

At this point, I waited for the speaker's multiple choice options. My thought was that it would be dominated by the ipc latency, since 
reading from memory was on the order of 100 ns, and ipc is a higher order of magnitude here. Post-hoc, I could additionally justify this choice 
by noting that Redis's GET operation is O(1). The options:

- 10 ms
- 1 ms
- 100 us
- 10 us
- 1 us
- < 1 us

I ended up picking 10us, because I wasn't sure if it would use unix domain sockets or tcp/udp loopback as the transport. But the answer was 
25 us, so I basically picked the correct order of magnitude.

If we didn't know the speed of memory access and IPC, it would have been much much more difficult to arrive at the correct answer. Therefore, 
it's essential that we know as many of these reference points for computers that we can manage, so that we can solve these types of problems.
It won't get us the whole way though; we also needed some qualitative understanding of Redis and its implementation in order to arrive at the 
correct value. This is why it's important to also read blog posts and understand the larger systems design ecosystem. 

So let's learn some reference points.

| Operation                           | Latency     | Throughput | 1 MiB  | 1 GiB  |
| ----------------------------------- | -------     | ---------- | ------ | ------ |
| Sequential Memory R/W (64 bytes)    | 0.5 ns      |            |        |        |
| ├ Single Thread, No SIMD            |             | 10 GiB/s   | 100 μs | 100 ms |
| ├ Single Thread, SIMD               |             | 20 GiB/s   | 50  μs | 50 ms  |
| ├ Threaded, No SIMD                 |             | 30 GiB/s   | 35  μs | 35 ms  |
| ├ Threaded, SIMD                    |             | 35 GiB/s   | 30  μs | 30 ms  |
| Network Same-Zone                   |             | 10 GiB/s   | 100 μs | 100 ms |
| ├ Inside VPC                        |             | 10 GiB/s   | 100 μs | 100 ms |
| ├ Outside VPC                       |             | 3 GiB/s    | 300 μs | 300 ms |
| Hashing, not crypto-safe (64 bytes) | 25 ns       | 2 GiB/s    | 500 μs | 500 ms |
| Random Memory R/W (64 bytes)        | 50 ns       | 1 GiB/s    | 1 ms   | 1s     |
| Fast Serialization `[8]` `[9]` †    | N/A         | 1 GiB/s    | 1 ms   | 1s     |
| Fast Deserialization `[8]` `[9]` †  | N/A         | 1 GiB/s    | 1 ms   | 1s     |
| System Call                         | 500 ns      | N/A        | N/A    | N/A    |
| Hashing, crypto-safe (64 bytes)     | 100 ns      | 1 GiB/s    | 1 ms   | 1s     |
| Sequential SSD read (8 KiB)         | 1 μs        | 4 GiB/s    | 200 μs | 200 ms |
| Context Switch `[1] [2]`            | 10 μs       | N/A        | N/A    | N/A    |
| Sequential SSD write, -fsync (8KiB) | 10 μs       | 1 GiB/s    | 1 ms   | 1s     |
| TCP Echo Server (32 KiB)            | 10 μs       | 4 GiB/s    | 200 μs | 200 ms |
| Decompression `[11]`                | N/A         | 1   GiB/s  | 1 ms   | 1s     |
| Compression `[11]`                  | N/A         | 500 MiB/s  | 2 ms   | 2s     |
| Sequential SSD write, +fsync (8KiB) | 1 ms        | 10 MiB/s   | 100 ms | 2 min  |
| Sorting (64-bit integers)           | N/A         | 200 MiB/s  | 5 ms   | 5s     |
| Sequential HDD Read (8 KiB)         | 10 ms       | 250 MiB/s  | 2 ms   | 2s     |
| Blob Storage GET, 1 conn            | 50 ms       | 500 MiB/s  | 2 ms   | 2s     |
| Blob Storage GET, n conn (offsets)  | 50 ms       | NW limit   |        |        |
| Blob Storage PUT, 1 conn            | 50 ms       | 100 MiB/s  | 10 ms  | 10s    |
| Blob Storage PUT, n conn (multipart)| 150 ms      | NW limit   |        |        |
| Random SSD Read (8 KiB)             | 100 μs      | 70 MiB/s   | 15 ms  | 15s    |
| Serialization `[8]` `[9]` †         | N/A         | 100 MiB/s  | 10 ms  | 10s    |
| Deserialization `[8]` `[9]` †       | N/A         | 100 MiB/s  | 10 ms  | 10s    |
| Proxy: Envoy/ProxySQL/Nginx/HAProxy | 50 μs       | ?          | ?      | ?      |
| Network within same region          | 250 μs      | 2 GiB/s    | 500 μs | 500 ms |
| Premium network within zone/VPC     | 250 μs      | 25 GiB/s   | 50 μs  | 40 ms  |
| {MySQL, Memcached, Redis, ..} Query | 500 μs      | ?          | ?      | ?      |
| Random HDD Read (8 KiB)             | 10 ms       | 0.7 MiB/s  | 2 s    | 30m    |
| Network between regions `[6]`       | [Varies][i] | 25 MiB/s   | 40 ms  | 40s    |
| Network NA Central <-> East         | 25 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network NA Central <-> West         | 40 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network NA East <-> West            | 60 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> NA East         | 80 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> NA Central      | 100 ms      | 25 MiB/s   | 40 ms  | 40s    |
| Network NA West <-> Singapore       | 180 ms      | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> Singapore       | 160 ms      | 25 MiB/s   | 40 ms  | 40s    |
_Taken from https://github.com/sirupsen/napkin-math._

Whoa! That's a lot. I don't know about you, but I don't really feel like memorizing this table. It's a lot of values that don't really 
mean much to me right now. We can get around this by learning the table incrementally -- we do a practice problem, looking up the numbers as we
need them, and then we add them to our Anki deck. We review our Anki deck every day, and hopefully we have it in our heads the next time we see
a similar problem.

My next step in my systems design journey is to in fact go through the problems accompanying sirupsen's talk, where problem n can be found at 
`https://sirupsen.com/napkin/problem-{n}`. I highly encourage you to watch it, and then go through these problems yourself. But that's all I 
have to say about systems design for now.
