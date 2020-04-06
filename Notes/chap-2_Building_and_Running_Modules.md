### Setting Up Your Test System

1. we recommend that you obtain a “mainline” kernel directly from the kernel.org mirror network, and install it on your system.
2. Vendor kernels can be heavily patched and divergent from the mainline; at times, vendor patches can change the kernel API as seen by device drivers. 
3. If you are writing a driver that must work on a particular distribution, you will certainly want to build and test against the relevant kernels.

### The Hello World Module

1. This module defines two functions, one to be invoked when the module is loaded into the kernel (hello_init)and one for when the module is removed (hello_exit). 
2. The module_init and module_exit lines use special kernel macros to indicate the role of these two functions. Another special macro (MODULE_LICENSE)is used to tell the kernel that this module bears a free license; without such a declaration, the kernel complains when the  module is loaded.
3. The module can call printk because, after insmod has loaded it, the module is linked to the kernel and can access the kernel’s public symbols (functions and variables, as detailed in the next section). 
4.  The string KERN_ALERT is the priority of the message. We’ve specified a high priority in this module, because a message with the default priority might not show up anywhere useful, depending on the kernel version you are running, the version of the klogd daemon, and your configuration.
5. The message goes to one of the system log files, such as /var/log/kern.log.

### User Space and Kernel Space
1. The role of the operating system, in practice, is to provide programs with a consistent view of the computer’s hardware. 
2. In addition, the operating system must account for independent operation of programs and protection against unauthorized
access to resources. This nontrivial task is possible only if the CPU enforces protection of system software from the applications.
3. The chosen approach is to implement different operating modalities (or levels)in the CPU itself. The levels have different roles, and some operations are disallowed at the lower levels.
4. All current processors have at least two protection levels, and some, like the x86 family, have more levels; when several levels exist, the highest and lowest levels are used. Under Unix, the kernel executes in the highest level (also called supervisor
mode), where everything is allowed, whereas applications execute in the lowest level (the so-called user mode), where the processor regulates direct access to hardware and unauthorized access to memory.
5. We usually refer to the execution modes as kernel space and user space. These terms encompass not only the different privilege levels inherent in the two modes, but also the fact that each mode can have its own memory mapping—its own address space—as well.

### Concurrency in the Kernel
1. can run on symmetric multiprocessor (SMP)systems, with the result that your driver could be executing concurrently on more than one CPU. 
2. Linux kernel code, including driver code, must be reentrant—it must be capable of running in more than one context at the same time.
3.  If you do not write your code with concurrency in mind, it will be subject to catastrophic failures that can be exceedingly difficult to debug.

### The Current Process
1.  Kernel code can refer to the current process by accessing the global item **current**, defined in **<asm/current.h>**, which yields a pointer to **struct task_struct**, defined by **<linux/sched.h>**.
2. The current pointer refers to the process that is currently executing.

### A Few Other Details
1. Applications are laid out in virtual memory with a very large stack area. The stack, of course, is used to hold the function call history and all automatic variables created by currently active functions.
2. The kernel, instead, has a very small stack; it can be as small as a single, 4096-byte page. Your functions must share that stack with the entire kernel-space call chain. Thus, it is never a good idea to declare large automatic variables; if you need larger structures, you should allocate them dynamically at call time.
3. Often, as you look at the kernel API, you will encounter function names starting with a double underscore (\__). Functions so marked are generally a low-level component of the interface and should be used with caution. Essentially, the double underscore says to the programmer: “If you call this function, be sure you know what you are doing.”
4. Kernel code cannot do floating point arithmetic. Enabling floating point would require that the kernel save and restore the floating point processor’s state on each entry to, and exit from, kernel space.

### Compiling Modules
1. The first is to ensure that you have sufficiently current versions of the compiler, module utilities, and other necessary tools. The file Documentation/Changes in the kernel documentation directory always lists the required tool versions; you
should consult it before going any further.
2. Trying to build a kernel (and its modules) with the wrong tool versions can lead to no end of subtle, difficult problems. Note
that, occasionally, a version of the compiler that is too new can be just as problematic as one that is too old; the kernel source makes a great many assumptions about the compiler, and new releases can sometimes break things for a while.


