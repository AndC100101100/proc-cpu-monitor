

"The `/proc` filesystem in Linux is a pseudo-filesystem that provides an interface to kernel data structures. Unlike regular filesystems that store data on disk, `/proc` exists purely in memory, exposing kernel and process information as files and directories."
> *— The Linux Kernel Documentation*

A BIG chunk of the code in this repo was made by one of the coolest persons on the internet, YSAP [You Suck At Programming](https://www.youtube.com/@yousuckatprogramming) as part of his bash learning series. Dave is a great content creator and a crazy good programmer, so I would advise if you want to learn bash and or programming, YSAP should be your go-to.

I liked the video and concept a lot. With that said, here's what I dug into after watching the series and poking around the code.

# Proc monitoring
This script's graphs and visualizations are able to be done thanks to the misc kernel statistics written into `/proc/stat`. This file contains plain text information about the kernel activity. In this case, the information written into it will always be an aggregate of numbers since the system first booted.

You can take a quick look at how this file looks with `cat /proc/stat`

You are going to see a few headers in this output file, all of which we use in this script to generate our CPU usage visualizations: 

## What each field really means:
We are going to take a quick look at some of these headers:

 user    - normal processes in user mode

 nice    - niced processes in user mode  

 system  - kernel mode time
 
 idle    - doing nothing

 iowait  - waiting for I/O 

 irq     - servicing hardware interrupts

 softirq - servicing software interrupts

 steal   - involuntary wait

 guest   - time spent running a normal guest OS

 guest_nice - time spent running niced guest
 
### What is up with iowait
In context of iowait, there are a few standing issues. For example, the CPU will not wait for I/O to complete, iowait is the time a task waits for I/O to complete. This means that when a CPU goes idle, it can schedule another task - the original task isn't blocking the CPU while waiting.
On the other hand, in a multi core CPU, the usual task waiting for the completion of this I/O isn't really running on any CPU, making it difficult to calculate the iowait and an unreliable metric, as we may have underestimations.

In a way, modern linux iowait is a bit different. We can make sense of it as the percentage of time the system was idle, in which at least one process was actually waiting for disk I/O to finish.

IOWait has its merits as a metric, but relying on it as the sole source of truth for performance evaluation would be misleading, it's just iowait is readily available on most linux systems.
 
iowait is still a steady solution to detect certain bottlenecks. Certain services may find themselves taking hits in performance while CPU usage stays normal. When it comes to databases, tools like `iostat` may show high `%util` and `await` times on the database's disk (/dev/sda), with high `%wa` (I/O wait) in vmstat. This could be representative of missing indexes when doing lookups or in the usage of complex queries, causing slowness in transactions.

A deeper dive is the only way to appropriately set a precedent, and IOWait may be the signal to our root problem for these types of issues. Common causes in databases may come from inefficient queries triggering excessive disk reads/writes, which may cause I/O bottlenecks. Many are the cases, and only then can we point to solutions like adding database indexes, optimizing queries, data caching and or, as a last resort, upgrading SSDs/NVMe drives.



## Some additions I did to the original script 

Used `/proc/uptime` adding extra UI stuff so it looks fancier.

Also used `/proc/loadavg` to create a small bar that takes on the load average to create a nice graph.

Some extra modifications include specific color based on the amount and or percentage of CPU being used.


## References:

- [The Linux Kernel Documentation - procstat](https://docs.kernel.org/filesystems/proc.html)

- [Decoding IOWait - Friend or Foe in System Performance](https://medium.com/@itshimanshusingh/decoding-iowait-friend-or-foe-in-system-performance-ed44c781b66e)

- [Debug Disk Performance with iostat and iotop](https://webdock.io/en/docs/mastering-web-fundamentals/server-fundamentals/debug-disk-performance-iostat-and-iotop)
