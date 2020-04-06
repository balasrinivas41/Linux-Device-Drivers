### The Role of the Device Driver
1. The role of a device driver is providing mechanism, not policy.
2. “what capabilities are to be provided” (the mechanism) and “how those capabilities can be
used” (the policy).
3. The floppy driver is policy free—its role is only to show the diskette as a continuous array of data blocks. Higher levels of the system provide policies, such as who may access the floppy drive, whether the drive is accessed directly or via a filesystem, and
whether users may mount filesystems on the drive.

### Splitting the Kernel
The kernel’s role can be split into the following parts:
- Process management
- Memory management
- Filesystems
- Device control
- Networking

### Loadable Modules
1. One of the good features of Linux is the ability to extend at runtime the set of features offered by the kernel. This means that you can add functionality to the kernel (and remove functionality as well) while the system is up and running.
2.Each piece of code that can be added to the kernel at runtime is called a module. The
Linux kernel offers support for quite a few different types (or classes) of modules, including, but not limited to, device drivers. Each module is made up of object code (not linked into a complete executable) that can be dynamically linked to the running kernel by the insmod program and can be unlinked by the rmmod program.
