---
layout: "../../layouts/BlogPost.astro"
title: "Memory Management in Linux"
description: ""
pubDate: "May 17 2023"
heroImage: "/placeholder-hero.jpg"
---

# Linux Memory Management Features

**1. Virtual Memory and Physical Memory**

Physical memory refers to the actual random-access memory (RAM) installed on a computer. Each byte of physical memory is associated with a physical address. However, processes running on a Linux system don't interact with physical memory directly. Instead, they work with an abstraction of physical memory known as virtual memory.

Virtual memory allows a process to believe it has its own private, large, and continuous block of memory. Each process is assigned its own virtual address space, which it uses as if it were the entire memory. Behind the scenes, the kernel and the hardware's memory management unit (MMU) map these virtual addresses to the actual physical addresses.

This has several benefits, including process isolation (one process cannot access another's memory) and the ability to use more memory than is physically available by swapping out to disk.

**2. Memory Allocation and Deallocation**

When a process is started, it makes a system call to request memory from the kernel. This can be done through various functions such as `malloc()` in C. The kernel then allocates a block of memory to the process, tracking the allocation in a data structure.

This memory comes from a pool of available memory managed by the kernel, which is divided into chunks of a fixed size known as pages. The kernel keeps track of which pages are currently allocated and which are free.

Throughout its lifecycle, a process may request additional memory or free up memory it no longer needs. When a process no longer needs a portion of memory, it can deallocate it, returning it to the kernel's pool of available memory.

When a process terminates, all memory allocated to it is automatically deallocated and returned to the pool of available memory.

**3. Paging and Swapping**

To manage memory more efficiently, Linux divides physical memory into fixed-size units called pages. This is part of the virtual memory system.

When physical memory becomes scarce, the kernel can move infrequently used pages to a special area on the disk known as swap space, effectively increasing the amount of available memory. This process is known as swapping or paging out.

When the data in these pages are needed again, they are read back into memory (this is called paging in). This allows Linux to use more memory than is physically available, at the cost of slower access times for pages that have been swapped out to disk.

**4. Memory Protection**

The kernel ensures that one process cannot access another process's memory. This is an important part of the system's security and stability.

Each process operates in its own virtual address space, and it cannot access memory outside of this space. If a process attempts to access memory that it hasn't been allocated, a segmentation fault occurs, and the process is terminated.

**5. /proc/meminfo**

The `/proc/meminfo` file provides a snapshot of the system's current memory usage. It's part of the proc filesystem, a pseudo-filesystem which is used as an interface to kernel data structures. The file includes various pieces of information about memory usage, including the total physical memory, the amount of free and used memory, the amount of memory used for buffers, and the total, used, and free swap space.

`MemAvailable` is an important field that was added in recent Linux kernel versions. It provides an estimate of how much memory is available for starting new applications, without swapping. This is calculated as the `MemFree` plus the caches and buffers that can be reclaimed, minus a low watermark value that the system tries to maintain for smooth operation.

**6. Garbage Collection**

The term "garbage collection" is often used to describe the process of automatically freeing up memory that is no longer in use. In

the context of the Linux kernel, when a process terminates or when it explicitly frees up memory that it no longer needs, that memory is returned to the pool of available memory.

The Linux kernel does not have a garbage collector in the way that some high-level languages like Java or Python do. Instead, it relies on explicit deallocation: when a process is done with a block of memory, it should release it back to the system. If a process terminates, all its memory is automatically reclaimed by the system.

`free` is a command in Linux that displays the total amount of free and used physical memory (RAM) in the system, as well as the swap space. The `free` command displays amounts in kilobytes (kB).

The `/proc/meminfo` file, on the other hand, provides more detailed information about the system's memory usage. This is a virtual filesystem provided by the kernel that contains system information. One of these files, `/proc/meminfo`, contains detailed information about the system's current memory usage.

## Free

Here's a sample output of the `free` command:

```
              total        used        free      shared  buff/cache   available
Mem:       16326428     6431876     4954072      225108     4935480     9148600
Swap:       2097148      132032     1965116

```

For the `Mem:` row:

1. `total = used + free + buff/cache`
2. `used = total - free - buff/cache`
3. `available = free + buff/cache - (part of the buff/cache that is not reclaimable)`

For the `Swap:` row:

1. `total = used + free`

Please note that the `available` memory is the amount of memory that can be reclaimed without swapping. It's calculated by the kernel and it takes into account that part of the buff/cache memory is not reclaimable because it's being used by the system to improve performance. It's a more accurate indicator of the memory available for starting new applications, without swapping.

Also, as a reminder, the `used` memory is different from `Active` memory in `/proc/meminfo`, because `used` also includes buffer and cache memory.

# /proc/meminfo

And here's a sample from the `/proc/meminfo` file:

```
MemTotal:       16326428 kB
MemFree:         4954072 kB
MemAvailable:    9148600 kB
Buffers:          242756 kB
Cached:          2356484 kB
SwapCached:        34300 kB
Active:          4542072 kB
Inactive:        2394844 kB
...
SwapTotal:       2097148 kB
SwapFree:        1965116 kB
...

```

Sure, here's how you can express the relation between different components in `/proc/meminfo`:

1. `MemTotal = MemFree + MemUsed`
2. `MemUsed = Active + Inactive + Buffers + Cached + Slab + (possibly others depending on the system)`
3. `MemAvailable = MemFree + Buffers + Cached + SReclaimable - Shmem`
4. `SwapTotal = SwapFree + SwapUsed`

The `MemUsed` is not a field you will directly find in `/proc/meminfo` but you can calculate it as `MemTotal - MemFree`.

`MemAvailable` is a little more complex. It's an estimate of how much memory is available for starting new applications, without swapping. The kernel calculates it by adding the memory that is free, that is used as buffers and cache, and that is reclaimable (like the Slab cache), and subtracting shared memory (Shmem).

The `SwapTotal` and `SwapFree` values are straightforward and indicate the total swap space and the free swap space. The used swap can be calculated as `SwapTotal - SwapFree`.

Please note that the exact calculation might vary depending on your specific Linux distribution and kernel version, and there may be other memory factors involved.

The `/proc/meminfo` file in Linux contains detailed information about the system's memory usage and configuration. The file provides a snapshot of the current state of the system's memory at the time it is read. Here is an overview of the contents of the `/proc/meminfo` file and what each entry means:

| Value          | Explanation                                                                                                                                                                             |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MemTotal       | Total amount of physical memory in kilobytes.                                                                                                                                           |
| MemFree        | Amount of physical memory currently unused in kilobytes.                                                                                                                                |
| MemAvailable   | Estimate of how much memory is available for starting new applications, without swapping. This value is calculated based on the memory usage of the system and can fluctuate over time. |
| Buffers        | Amount of memory used by kernel buffers in kilobytes.                                                                                                                                   |
| Cached         | Amount of memory used by kernel caches in kilobytes. This includes both file system caches and page caches.                                                                             |
| SwapCached     | Amount of memory in the swap cache in kilobytes.                                                                                                                                        |
| Active         | Amount of memory currently in use, including memory that is shared between multiple processes, in kilobytes.                                                                            |
| Inactive       | Amount of memory that is not currently in use and has been marked as available for allocation in kilobytes.                                                                             |
| Active(anon)   | Amount of anonymous memory currently in use in kilobytes.                                                                                                                               |
| Inactive(anon) | Amount of anonymous memory that is not currently in use in kilobytes.                                                                                                                   |
| Active(file)   | Amount of memory used by file-backed mappings that is currently in use in kilobytes.                                                                                                    |
| Inactive(file) | Amount of memory used by file-backed mappings that is not currently in use in kilobytes.                                                                                                |
| Unevictable    | Amount of memory that cannot be freed due to being locked in memory in kilobytes.                                                                                                       |
| Mlocked        | Amount of memory that has been locked in memory using mlock() in kilobytes.                                                                                                             |
| SwapTotal      | Total amount of swap space available in kilobytes.                                                                                                                                      |
| SwapFree       | Amount of unused swap space in kilobytes.                                                                                                                                               |
| Dirty          | Amount of memory waiting to be written back to disk in kilobytes.                                                                                                                       |
| Writeback      | Amount of memory actively being written back to disk in kilobytes.                                                                                                                      |
| AnonPages      | Amount of anonymous memory (memory not associated with a file) used by processes in kilobytes.                                                                                          |
| Mapped         | Amount of memory used for file mappings in kilobytes.                                                                                                                                   |
| Shmem          | Amount of shared memory (shmem) in use in kilobytes.                                                                                                                                    |
| Slab           | Amount of memory used by kernel data structures (slab allocator) in kilobytes.                                                                                                          |
| SReclaimable   | Amount of slab memory that is reclaimable in kilobytes.                                                                                                                                 |
| SUnreclaim     | Amount of slab memory that is not reclaimable in kilobytes.                                                                                                                             |
| KernelStack    | Amount of kernel stack memory used by each active process in kilobytes.                                                                                                                 |
| PageTables     | Amount of memory used by page table structures in kilobytes.                                                                                                                            |
| NFS_Unstable   | Amount of memory used by NFS filesystems in kilobytes.                                                                                                                                  |
| Bounce         | Amount of memory used by the buffer bounce pool in kilobytes.                                                                                                                           |
| WritebackTmp   | Amount of memory used by the kernel's tmpfs implementation for storing dirty data waiting to be written back to disk in kilobytes.                                                      |
| CommitLimit    | Total amount of memory that can be allocated to processes without causing swapping, calculated as the sum of physical memory and swap space in kilobytes.                               |
| Committed_AS   | Estimated amount of memory that has been allocated to processes by the kernel in kilobytes.                                                                                             |
| VmallocTotal   | Total amount of memory available for virtual allocations in kilobytes.                                                                                                                  |
| VmallocUsed    | Amount of memory currently used for virtual allocations in kilobytes.                                                                                                                   |
| VmallocChunk   | Largest contiguous block of virtual memory available for allocation in kilobytes                                                                                                        |

Now, let's map the values:

- `MemTotal` in `/proc/meminfo` corresponds to the `total` column of the `Mem:` row in `free` output.
- `MemFree` corresponds to the `free` column of the `Mem:` row.
- `Buffers` and `Cached` (excluding `SwapCached`) in `/proc/meminfo` together correspond to `buff/cache` in `free` output.
- `MemAvailable` in `/proc/meminfo` corresponds to the `available` column of the `Mem:` row.
- `SwapTotal` in `/proc/meminfo` corresponds to the `total` column of the `Swap:` row in `free` output.
- `SwapFree` corresponds to the `free` column of the `Swap:` row.

It's worth mentioning that the `used` memory in `free` command is calculated as `total - free - buffers - cache`. This means the used memory shown by `free` might not match `Active` memory in `/proc/meminfo`, because `free` also considers buffer and cache memory as used.

Please note that the exact matching might vary depending on the specific Linux distribution and kernel version.

## Resident Set Size (RSS) vs. Unique Set Size (USS) vs. Proportional Set Size (PSS)

When it comes to understanding the memory usage of a process, there are several important metrics. Among these are the Resident Set Size (RSS), Unique Set Size (USS), and Proportional Set Size (PSS).

1. **Resident Set Size (RSS):** RSS is the portion of a process's memory that is held in RAM. This includes all stack, heap, and other non-swappable memory segments used by the process. Note that this also includes memory shared with other processes (such as shared libraries). If several processes use the same shared library, the memory used by that library will be counted in full in the RSS of each of the processes. This means that the total RSS of all processes can exceed the total physical memory.
2. **Unique Set Size (USS):** USS is the portion of a process's memory that is held in RAM and not shared with any other process. It is the memory that would be freed if the process was terminated right now. USS is useful for understanding the memory footprint of a single process, but it doesn't account for shared resources, so summing the USS values of multiple processes will undercount any shared memory.
3. **Proportional Set Size (PSS):** PSS is a measurement of a process's memory that is held in RAM, similar to RSS. However, unlike RSS, PSS counts shared libraries proportionally among the processes that use them. For example, if there are three processes each using a shared library that takes up 30 KB of memory, that library would add 30 KB to the RSS of each process (for a total of 90 KB), but only 10 KB to the PSS of each process (for a total of 30 KB). PSS is a more accurate measurement for understanding the memory footprint of processes in a system where shared libraries are common.

In summary, while RSS can overestimate the amount of memory used by processes due to counting shared memory multiple times, USS can underestimate it by not accounting for shared memory at all. PSS, on the other hand, provides a balanced view by proportionally attributing shared memory to the processes that use it.

### Finding USS/PSS of a process

`smem` is a tool that provides numerous reports on memory usage on Linux, including a report that includes USS (Unique Set Size). USS is the unshared memory unique to a process - this is the memory that would be freed if the process were to terminate.

Firstly, you need to have `smem` installed on your system. If it's not, you can install it via your system's package manager. For example, on Ubuntu, you can install it using `apt`:

```
sudo apt-get install smem

```

Once you have `smem` installed, you can use it to calculate USS for a process. Simply run `smem` with no arguments to get a report on all processes:

```
smem
```

This will provide an output that looks something like this:

```
  PID User     Command                         Swap      USS      PSS      RSS
 1234 user     /usr/lib/some-process              0     1234     1400     2000
 1235 user     /usr/bin/another-process           0     2345     2500     3000
```

The `USS` column in this report shows the Unique Set Size for each process.

If you're interested in a specific process, you can use the `-r` option and `grep` to filter for that process. For example, to get the USS for processes named `some-process`, you could use:

```
smem -r | grep some-process
```

This will give you the USS along with other memory statistics for all processes named `some-process`.

## Shared Memory

Shared memory, as the name suggests, is a portion of memory that can be accessed by multiple processes. It is typically used when different processes need to communicate with each other or share data.

If a process that uses shared memory terminates, the shared memory it was using is not immediately reclaimed. This is because other processes might still be using that shared memory. The kernel keeps track of the number of processes that are using a shared memory segment. Only when this count drops to zero (i.e., no process is using the shared memory) can the shared memory be reclaimed.

This behavior is important to ensure that one process terminating does not inadvertently impact other processes that are still using the shared memory. However, it also means that it's possible for shared memory to be "leaked" if processes don't properly detach from it when they're done using it.

There are tools and system calls available in Linux to manage shared memory, including creating shared memory segments, attaching or detaching them from a process's address space, and controlling their permissions. It's the responsibility of the developer to use these tools correctly to ensure shared memory is used efficiently and cleaned up when no longer needed.

## Memory Leaks

A memory leak occurs when a program allocates memory but fails to free it when it's no longer needed. Over time, this can consume more and more memory, slowing down the system or even causing it to run out of memory. Here are some ways to diagnose a memory leak in Linux:

**1. Monitor System Memory Usage:** Tools such as `top`, `htop`, `free`, and `vmstat` can provide information about overall system memory usage. If you notice that memory usage continually increases over time, especially when it doesn't decrease even when the workload decreases, that might indicate a memory leak.

**2. Identify Problematic Processes:** `top` and `ps` are useful for identifying which processes are using the most memory. With `top`, you can press `Shift + M` to sort processes by memory usage. With `ps`, you can use `ps aux --sort -rss` to sort processes by RSS (Resident Set Size), which represents the portion of the process's memory held in RAM.

**3. Use Valgrind:** Valgrind is a powerful tool for memory debugging, memory leak detection, and profiling. The Memcheck tool within Valgrind can be particularly useful for finding memory leaks in your programs. It can detect memory that is allocated but not freed and point out where in the code the memory was allocated.

**4. Use Debugging Tools:** Tools like gdb (GNU Debugger) can also help diagnose memory leaks, especially if you have the source code of the program.

**5. Use Language-Specific Tools:** If you're working with a high-level language, there may be tools specific to that language for diagnosing memory leaks. For example, Python has `objgraph` and `gc` module, JavaScript has development tools in browsers, Java has VisualVM, etc.

**6. Analyze Core Dumps:** If your application is crashing due to running out of memory, you can analyze the core dump using gdb or a similar tool to see what was in memory at the time of the crash.

Remember that diagnosing memory leaks can be complex, especially in large applications or when the leak is small. It often involves a combination of monitoring, testing, and code analysis. It's also worth noting that prevention is better than cure - good coding practices can help prevent memory leaks in the first place.

## Swapping

Swapping occurs when the physical memory (RAM) of a system is nearly full, and the system begins moving pages of memory over to swap space to prevent a complete memory outage. Increased swapping can be indicative of your system not having enough physical memory to accommodate the tasks it's being asked to perform. To diagnose the cause of increased swapping, you can use several tools and techniques:

1. **Check the Swap Usage:** The first step is to check the swap usage on your system. You can use the `free -m` or `swapon -s` commands in Linux to check the amount of swap space used and available.
2. **Identify High Memory Usage Processes:** Use the `top` or `htop` command to list processes and their resource usage. With `top`, you can press `Shift + M` to sort processes by memory usage. Check if any processes are using an unusually high amount of memory.
3. **Check for Memory Leaks:** If a process continually increases its memory usage over time, that could be indicative of a memory leak. Tools like `valgrind` can help diagnose memory leaks in programs.
4. **Monitor Over Time:** The `vmstat` command can provide reports about memory, swap, I/O, and CPU activity. Running `vmstat 5` will provide an update every 5 seconds. The `si` and `so` columns show the amount of memory swapped in from disk and swapped out to disk respectively. If these numbers are consistently above 0, your system is actively swapping.
5. **Check Swappiness Value:** The swappiness value determines how aggressively the system will swap memory pages versus dropping pages from the page cache. A higher value increases the likelihood of swapping, while a lower value tells the system to swap only to avoid an out of memory condition. You can check the swappiness value by running `cat /proc/sys/vm/swappiness`. The default value is typically 60. If needed, you can adjust the swappiness value temporarily using the `sysctl` command or permanently by modifying the `/etc/sysctl.conf` file.
6. **Evaluate System Memory Requirements:** If your system is consistently using a large amount of swap, it may simply be that you don't have enough physical memory to handle your workload. You may need to close unnecessary applications or services, optimize your applications to use less memory, or install more RAM.

Remember, some degree of swapping is normal, but excessive swapping, known as "swap thrashing," can significantly degrade system performance as read and write operations to a hard drive are much slower than to RAM.
