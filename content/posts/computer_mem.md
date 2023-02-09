---
title: "Notes from \"What every programmer should know about memory\""
date: 2023-01-10T16:03:23+01:00
draft: true
---

## Intro

Here is the document ["What every programmer should know about memory."](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) written by Urlich Drepper in 2007.

I know I'm making notes of notes, but it helps me. I won't really be covering the first few chapters because I don't want to.

## First chapters

Talking about SRAM and DRAM. SRAM is expensive but faster, DRAM is cheap but slower (capacitor makes things slower). In consummer electronics DRAM wins.

## Caching 

We can use a bit of SRAM as a cache. Because we have [spatial and temporal locality](https://en.wikipedia.org/wiki/Locality_of_reference), we can cache a small dataset and get a high percentage of cache hits.

He says

> The memory regions needed for code and data are pretty much independent of each other, which is why independent caches work better

From this I understand that we seperate the caches for code and data because nearby code and data in memory are generally uncorrelated. So spatial locality does not apply.

Since DRAM modules can read consecutive memory cells much faster by holding the row-address-selection signal high, we can load in cache entire "lines" consisting of multiple words and exploit spatial locality.

Say a cache line is 8 x 64-bit words, so 64 bytes. To index each cache line we have:

- 6 bits reserved to offset into the cache line, since \\(2^6 = 64\\), so we can look through the cache line byte by byte.
- The remaining \\(32 - 6 = 26\\) bits are to identify the cache line.

A cache line which has been written to without the write being reflected in main memory is called 'dirty'.

A cache miss has roughly the following penalty.

| Memory | Cycles |
| ------ | ------ |
| Register | \\(\leq 1\\) |
| L1d | \\(\sim 3\\) |
| L2 | \\(\sim 14\\) |
| Main memory | \\(\sim 240\\) |

Interestingly, a large part of the access time for L2 is caused by wire delays/signal propogation. A larger cache necessitates longer wires.

A *fully associative* cache allows each tag to map to any cache entry. But this is costly to implement (many transistors) when cache size is over a few dozen entries.

A *direct-mapped* cache restricts each tag to a single cache entry. If some memory addresses are cached more frequently than others, then the corresponding cache entries will be evicted more frequently. This means each cache entry is not used evenly and can be quite a drawback.

A *set-associative* cache restricts each tag to map to a set of cache entries. It acts as a middle-ground between the fully associative and direct-mapped caches.

