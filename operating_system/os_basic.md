操作系统

操作系统是能够抽象底层不同种类的硬件设备为上层运行的多个进程提供访问这些设备还有系统其它资源的统一接口的程序。其中类Unix接口是非常知名的操作系统软件接口。被许多其它知名操作系统所借鉴采纳。

操作系统通常总体可以有两部分组成，一部分是内核程序，即kernel，对外暴露系统调用接口。另一部分是用户程序，也可以成为进程，操作系统为每一个进程提供资源，包括内存，程序，堆栈。

这两个部分在运行时是具备不同的权限的，我们也称作他们运行在不同的模式下，内核程序运行在内核态，具有直接操作底层硬件资源的能力。而用户程序则运行在用户态下面，没有内核态的高权限。

这种权限控制是通过使用CPU的硬件保护机制来实现的（也就是说权限控制是硬件实现的？）


