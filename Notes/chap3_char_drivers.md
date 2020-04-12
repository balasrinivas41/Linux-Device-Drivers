# Char Drivers
The goal of this chapter is to write a complete char device driver. We develop a character driver because this class is suitable for most simple hardware devices. Char drivers are also easier to understand than block drivers or network drivers
### The Design of scull
1.  scull (Simple Character Utility for Loading Localities). scull is a char driver that acts on a memory area as though it were a device. In this chapter, because of that peculiarity of scull, we use the word device interchangeably with “the memory area used by scull.”
2. The advantage of scull is that it isn’t hardware dependent. scull just acts on some memory, allocated from the kernel. Anyone can compile and run scull, and scull is portable across the computer architectures on which Linux runs. On the other hand, the device doesn’t do anything “useful” other than demonstrate the interface between the kernel and char drivers and allow the user to run some tests.
3. The first step of driver writing is defining the capabilities (the mechanism) the driver will offer to user programs.. It can be a sequential or random-access device, one device or many, and so on.
4. **scull0 to scull3**
- Four devices, each consisting of a memory area that is both global and persistent. Global means that if the device is opened multiple times, the data contained within the device is shared by all the file descriptors that opened it. Persistent means that if the device is closed and reopened, data isn’t lost. This device can be fun to work with, because it can be accessed and tested using conventional commands, such as cp, cat, and shell I/O redirection.
5. **scullpipe0 to scullpipe3**
- Four FIFO (first-in-first-out) devices, which act like pipes. One process reads what another process writes. If multiple processes read the same device, they contend for data. The internals of scullpipe will show how blocking and nonblocking read and write can be implemented without having to resort to interrupts. Although real drivers synchronize with their devices using hardware interrupts, the topic of blocking and nonblocking operations is an important one and is separate from interrupt handling.
6. **scullsingle**
- **scullpriv**
- **sculluid**
- **scullwuid**
7. These devices are similar to scull0 but with some limitations on when an open is permitted. The first (scullsingle) allows only one process at a time to use the driver, whereas scullpriv is private to each virtual console (or X terminal session), because processes on each console/terminal get different memory areas. sculluid and scullwuid can be opened multiple times, but only by one user at a
time; the former returns an error of “Device Busy”.
8. Each of the scull devices demonstrates different features of a driver and presents different difficulties.
### Major and Minor Numbers
1. Char devices are accessed through names in the filesystem. Those names are called special files or device files or simply nodes of the filesystem tree; they are conventionally located in the /dev directory. Special files for char drivers are identified by a “c”
in the first column of the output of ls –l. Block devices appear in /dev as well, but they are identified by a “b.”
2. If you issue the ls –l command, you’ll see two numbers (separated by a comma) in the device file entries before the date of the last modification, where the file length normally appears. These numbers are the major and minor device number for the particular device. 
3. Traditionally, the major number identifies the driver associated with the device.
4. Modern Linux kernels allow multiple drivers to share major numbers, but most devices that you will see are still organized on the
one-major-one-driver principle. The minor number is used by the kernel to determine exactly which device is being referred to. Depending on how your driver is written (as we will see below), you can either get a direct pointer to your device from the kernel, or you can use the minor number yourself as an index into a local array of devices. Either way, the kernel itself knows almost nothing about minor numbers beyond the fact that they refer to devices implemented by your driver.

### The Internal Representation of Device Numbers
1. Within the kernel, the dev_t type (defined in <linux/types.h>) is used to hold device numbers—both the major and minor parts. As of Version 2.6.0 of the kernel, dev_t is a 32-bit quantity with 12 bits set aside for the major number and 20 for the minor number.
2.  Your code should, of course, never make any assumptions about the internal organization of device numbers; it should, instead, make use of a set of macros found in <linux/kdev_t.h>. To obtain the major or minor parts of a dev_t, use:
- **MAJOR(dev_t dev);**
- **MINOR(dev_t dev);**
3. If, instead, you have the major and minor numbers and need to turn them into a dev_t,
use:
- **MKDEV(int major, int minor);**

### Allocating and Freeing Device Numbers
1. One of the first things your driver will need to do when setting up a char device is to obtain one or more device numbers to work with. The necessary function for this task is register_chrdev_region, which is declared in <linux/fs.h>:
- **int register_chrdev_region(dev_t first, unsigned int count, char *name);**
2. Here, first is the beginning device number of the range you would like to allocate. The minor number portion of first is often 0, but there is no requirement to that effect. count is the total number of contiguous device numbers you are requesting. Note that, if count is large, the range you request could spill over to the next major number; but everything will still work properly as long as the number range you request is available. Finally, name is the name of the device that should be associated with this number range; it will appear in /proc/devices and sysfs.
3. As with most kernel functions, the return value from register_chrdev_region will be 0 if the allocation was successfully performed. In case of error, a negative error code will be returned, and you will not have access to the requested region.register_chrdev_region works well if you know ahead of time exactly which device numbers you want. Often, however, you will not know which major numbers your
device will use; there is a constant effort within the Linux kernel development community to move over to the use of dynamicly-allocated device numbers. The kernel will happily allocate a major number for you on the fly, but you must request this allocation by using a different function:
- **int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);**
4. With this function, dev is an output-only parameter that will, on successful completion, hold the first number in your allocated range. firstminor should be the requested first minor number to use; it is usually 0. The count and name parameters work like those given to request_chrdev_region.
5. Regardless of how you allocate your device numbers, you should free them when they are no longer in use. Device numbers are freed with:
- **void unregister_chrdev_region(dev_t first, unsigned int count);**
6. The usual place to call unregister_chrdev_region would be in your module’s cleanup function.
