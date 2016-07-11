java-limit-memory
==================
java-limit-memory是一个在docker容器中限制java程序使用内存的脚本。


Backgroud
=========
启动docker容器时用-m参数指定了内存限制，但是java进程中获取的内存任然是宿主机的内存。这就导致启动时没有指定最大内存参数的java程序，默认的内存有可能超出容器内存限制，而被docker daemon进程kill。


Synopsis
========
java8的内存占用由堆内存(Heap)，元空间(MetaSpace)，直接内存(DirectMemory)3部分组成。java程序可以在启动参数中设置-Xmx/-XX:MaxHeapSize, -XX:MaxMetaspaceSize, -XX:MaxDirectMemorySize这些参数来限制各部分内存的大小。docker容器的内存限制可以从/sys/fs/cgroup/memory/memory.limit_in_bytes文件中获取。java-limit-memory脚本会根据docker容器的内存限制，与java的运行参数，动态计算出java堆内存的大小。





ReservedMemory
==============
有些情况下，并不希望java程序用完整个容器的内存。我们可以通过环境变量ReservedMemory来设置保留内存。java-limit-memory会先从容器的内存限制中减去ReservedMemory部分，然后再动态计算大小。
如果不指定ReservedMemory，默认值为64M。


Installation
============
执行java-limit-memory-installer脚本，可以将java-limit-memory脚本安装到系统中。
