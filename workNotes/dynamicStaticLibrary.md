# 动态库：

1. 要做成so的文件保存成两个文件，分别为头文件so.h和程序文件so.c，注意：c中要包含引用h，头文件包含不能用<>,而要用“”（因为前者只搜索系统路径，不会搜索用户路径）；

2. 使用指令：

    gcc so.c -fPIC -shared -o libtest.so

3. 上一步已经生成libtest.so动态库，然后就可以使用了，例如tcp_client.c需要用到该动态库，则指令为：

    gcc tcp_client.c -L. -ltest -o test

4. 随即test可执行文件就生成好了，用ldd test查看动态库是否链接上了，答案是肯定没有，要把libtest.so拷贝到系统的/lib目录下！


# 静态库：

1. gcc -c （生成.o文件so.o）

2. ar crs libhello.a so.o （生成静态库文件）

3. gcc tcp_client.c -L. -lhello即可


# 备注：

1. -fPIC表示生成位置无关的代码，如果直接由C文件生成so文件则直接放在由C文件生成so文件的makefile语句里即可，但如果先由C文件生成o文件，再由o文件生成so文件时，则该参数必须放在由C文件生成o文件的makefile语句里，因为该参数作用于编译阶段，而-shared则放在由o文件生成so文件的链接阶段；

2. GCC编译选项中有 -I（大写i）-L（大写l） -l（小写l）三种参数，例如：

    gcc -o hello hello.c -I/home/hello/include -L/home/hello/lib -lworld，其中
    
    -I/home/hello/include表示将该目录作为第一个寻找头文件的目录，需找顺序为：/home/hello/include --> /usr/include --> /usr/local/include，注意该参数用于编译阶段
    
    -L/home/hello/lib表示将该目录作为第一个寻找库文件的目录，寻找顺序为：/home/hello/lib --> /lib --> /usr/lib --> /usr/local/lib，注意该参数用于链接阶段
    
    -lworld表示在上面的lib路径中寻找libworld.so动态库文件（如果选项中加入-static表示寻找静态库），注意该参数用于链接阶段
