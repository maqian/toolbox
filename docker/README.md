文件列表
=======
java-limit-memory，java程序启动脚本，容器中使用此脚本来启动java程序，可以给java进程加上默认的内存限制
java-limit-memory-installer，这是一个安装脚本，用于将java-limit-memory脚本安装到镜像中


背景介绍
=======
Docker容器启动时可使用-m参数限制内存，在当前版本的Java虚拟机中，不能正确读取Docker容器限制的内存，仍然会使用宿主机的内存。如果Java进程启动时没有指定内存参数，则会使用默认的值（默认情况下，使用总内存的1/64作为初始堆内存，使用总内存的1/4作为最大堆内存），这样很容易导致容器中的Java进程在运行过程中申请超出容器限制的内存。根据策略，如果Docker Daemon检测到容器使用的内存超出了限制，会直接将容器kill。
为了保证容器中的Java进程只使用容器限制的内存，必须在java启动参数中指定内存大小。为了避免没有设置java内存参数导致的内存超限问题，脚本中给java启动参数强制加上了默认的内存参数，并使用断言保证java进程只使用容器限制的内存。


工作原理
=======
java-limit-memory会检测java启动参数，对没有设置的内存参数指定默认值。流程如下：
1. 从文件/sys/fs/cgroup/memory/memory.limit_in_bytes读取cgroups内存限制LimitMemory
1. 检测是否含有环境变量ReservedMemorySize（这个变量为脚本预设，非jvm标准参数，用于在容器中保留non-heap内存给线程栈使用），没有则设置为60M
1. 检测是否含有参数-XX:MaxMetaspaceSize，没有则设置为100M
1. 检测是否含有参数-XX:MaxDirectMemorySize，没有则设置为32M
1. 检测是否含有参数-XX:ReservedCodeCacheSize，没有则设置为100M
1. 检测是否含有参数-XX:CompressedClassSpaceSize，没有则设置为32M
1. 检测是否含有参数-XX:MaxHeapSize或者-Xmx，没有则设置为MaxHeapSize = LimitMemory - MaxThreadStackSize - MaxMetaspaceSize - MaxDirectMemorySize - ReservedCodeCacheSize - CompressedClassSpaceSize
1. 检测是否含有参数-Xms，没有则设置为MaxHeapSize/2
1. 检测是否含有-Xss参数，没有则设置为256K


如何安装
=======
运行安装脚本
`curl -sSL 'https://raw.githubusercontent.com/maqian/workarounds/master/docker/java-limit-memory-installer' | sh`
安装脚本执行完后，会将系统的java程序文件重命名为system-java，将java-limit-memory脚本保存为java文件

