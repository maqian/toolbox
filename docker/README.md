文件列表
=======
java-limit-memory，java程序启动脚本，容器中使用此脚本来启动java程序，可以给java进程加上默认的内存限制
java-limit-memory-installer，这是一个安装脚本，用于将java-limit-memory脚本安装到目标环境中


背景介绍
=======
Docker容器启动时可使用-m参数限制内存，在当前版本的Java虚拟机中，不能正确读取Docker容器限制的内存，仍然会使用宿主机的内存。如果Java进程启动时没有指定内存参数，则会使用默认的值（默认情况下，使用总内存的1/64作为初始堆内存，使用总内存的1/4作为最大堆内存），这样很容易导致容器中的Java进程在运行过程中申请超出容器限制的内存。如果Docker Daemon检测到容器使用的内存超出了限制，会直接将容器kill。
为了保证容器中的Java进程只使用容器限制的内存，必须在java启动参数中指定内存大小。为了避免没有设置java内存参数导致的内存超限问题，脚本中给java启动参数强制加上了默认的内存参数，并使用断言保证java进程只使用容器限制的内存。


工作原理
=======
java-limit-memory会检测java启动参数，对没有设置的内存参数指定默认值。流程如下：
1. 从文件/sys/fs/cgroup/memory/memory.limit_in_bytes读取cgroups内存限制LimitMemory
1. 检测是否含有环境变量X_HEAP_RATIO，如果没有则设置为0.6
1. 检测是否含有JAVA启动参数-XX:MaxHeapSize或者-Xmx，没有则设置为MaxHeapSize = LimitMemory * X_HEAP_RATIO
1. 检测是否含有JAVA启动参数-Xms，没有则设置为MinHeapSize = MaxHeapSize / 2
1. 检测是否含有JAVA启动参数-Xss，没有则设置为512K
1. 检测是否含有JAVA启动参数-XX:ParallelGCThreads，没有则设置为2
1. 检测是否含有JAVA启动参数-XX:MinHeapFreeRatio，没有则设置为20
1. 检测是否含有JAVA启动参数-XX:MaxHeapFreeRatio，没有则设置为40
1. 检测是否含有JAVA启动参数-XX:GCTimeRatio，没有则设置为4
1. 检测是否含有JAVA启动参数-XX:AdaptiveSizePolicyWeight，没有则设置为90
1. 检测环境变量X_INT，如果值为on|ON，则添加JAVA启动参数-Xint
1. 检测环境变量X_TRACE，如果只为on|ON，则添加JAVA启动参数-XX:+PrintVMOptions -XX:+PrintCommandLineFlags -XX:+UnlockDiagnosticVMOptions -XX:NativeMemoryTracking=summary -XX:+PrintNMTStatistics

如何安装
=======
运行安装脚本
`curl -sSL 'https://raw.githubusercontent.com/maqian/workarounds/master/docker/java-limit-memory-installer' | sh`
安装脚本执行完后，会创建/usr/local/bin/java脚本文件，运行/usr/local/bin/java脚本会调用系统的java程序。

