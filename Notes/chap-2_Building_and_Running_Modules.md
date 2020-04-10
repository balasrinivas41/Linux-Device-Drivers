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

### Loading and Unloading Modules
1. insmod accepts a number of command-line options (for details, see the manpage), and it can assign values to parameters in your module
before linking it to the current kernel. Thus, if a module is correctly designed, it can be configured at load time; load-time configuration gives the user more flexibility than compile-time configuration, which is still used sometimes.
2. It relies on a system call defined in kernel/module.c. The function sys_init_module allocates kernel memory to hold a module (this memory is allocated with vmalloc; see the section “vmalloc and Friends” ; it then copies the module text into that memory region, resolves kernel references in the module via the kernel symbol table, and calls the module’s initialization function to get everything going.
3. modprobe, like insmod, loads a module into the kernel. It differs in that it will look at the module to be loaded to see
whether it references any symbols that are not currently defined in the kernel. If any such references are found, modprobe looks for other modules in the current module search path that define the relevant symbols. When modprobe finds those modules (which are needed by the module being loaded), it loads them into the kernel as well. If you use insmod in this situation instead, the command fails with an “unresolved symbols” message left in the system logfile. 
4. The lsmod program produces a list of the modules currently loaded in the kernel. Some other information, such as any other modules making use of a specific module, is also provided. lsmod works by reading the /proc/modules virtual file.

## Version Dependency
1. The kernel does not just assume that a given module has been built against the proper kernel version. One of the steps in the build process is to link your module against a file (called vermagic.o)from the current kernel tree; this object contains a fair amount of information about the kernel the module was built for, including the target kernel version, compiler version, and the settings of a number of important configuration variables. When an attempt is made to load a module, this information can be tested for compatibility with the running kernel. If things don’t match, the module is not loaded.
2. If you are writing a module that is intended to work with multiple versions of the kernel (especially if it must work across major releases), you likely have to make use of macros and #ifdef constructs to make your code build properly.
3. **UTS_RELEASE**This macro expands to a string describing the version of this kernel tree. For
example, "2.6.10".
4. **LINUX_VERSION_CODE** This macro expands to the binary representation of the kernel version, one byte for each part of the version release number. For example, the code for 2.6.10 is 132618 (i.e., 0x02060a).* With this information, you can (almost)easily determine what version of the kernel you are dealing with.
5. **KERNEL_VERSION(major,minor,release)** This is the macro used to build an integer version code from the individual numbers that build up a version number. For example, KERNEL_VERSION(2,6,10) expands to 132618. This macro is very useful when you need to compare the
current version and a known checkpoint.

## Platform Dependency
1. the IA32 (x86)architecture has been subdivided into several different processor types. The old 80386 processor is still supported (for now), even though its instruction set is, by modern standards, quite limited. The more modern processors in this architecture have introduced a number of new capabilities, including faster instructions for entering the kernel, interprocessor locking, copying data, etc.
2. Newer processors can also, when operated in the correct mode, employ 36-bit physical addresses, allowing them to address more than 4 GB of physical memory. Other processor families have seen similar improvements.
3. The kernel, depending on various configuration options, can be built to make use of these additional features. Clearly, if a module is to work with a given kernel, it must be built with the same understanding of the target processor as that kernel was. Once again, the vermagic.o object comes in to play. When a module is loaded, the kernel checks the processorspecific configuration options for the module and makes sure they match the running kernel. If the module was compiled with different options, it is not loaded.
## The Kernel Symbol Table
1. The table contains the addresses of global kernel items—functions and variables—that are needed to implement modularized drivers. When a module is loaded, any symbol exported by the module becomes part of the kernel symbol table. 
2. New modules can use symbols exported by your module, and you can stack new modules on top of other modules. Module stacking is implemented in the mainstream kernel sources as well: the msdos filesystem relies on symbols exported by the fat module, and each input USB device module stacks on the usbcore and input modules.
> **EXPORT_SYMBOL(name);**
> **EXPORT_SYMBOL_GPL(name);**
3. The _GPL version makes the symbol available to GPL-licensed modules only. Symbols must be exported in the global part of the module’s file, outside of any function, because the macros expand to the declaration of a special-purpose variable that is expected to be accessible globally. This variable is stored in a special part of the module executible (an “ELF section”)that is used by the kernel at load time to find the variables exported by the module. 

## Preliminaries
- #include <linux/module.h>
- #include <linux/init.h>
1. module.h contains a great many definitions of symbols and functions needed by loadable modules. You need init.h to specify your initialization and cleanup functions.
2. It is not strictly necessary, but your module really should specify which license applies to its code. Doing so is just a matter of including a MODULE_LICENSE line:
MODULE_LICENSE("GPL");
3. The specific licenses recognized by the kernel are “GPL” (for any version of the GNU General Public License), “GPL v2” (for GPL version two only), “GPL and additional rights,” “Dual BSD/GPL,” “Dual MPL/GPL,” and “Proprietary.”
4. Other descriptive definitions that can be contained within a module include MODULE_AUTHOR (stating who wrote the module), MODULE_DESCRIPTION (a human-readable statement of what the module does), MODULE_VERSION (for a code revision number; see the comments in <linux/module.h> for the conventions to use in creating version strings), MODULE_ALIAS (another name by which this module can be known),
and MODULE_DEVICE_TABLE (to tell user space about which devices the module supports).
5. The various MODULE_ declarations can appear anywhere within your source file outside of a function. A relatively recent convention in kernel code, however, is to put these declarations at the end of the file.
## Initialization and Shutdown
### static int \__init initialization_function(void)
### {
 ### /* Initialization code here */
### }
### module_init(initialization_function);
