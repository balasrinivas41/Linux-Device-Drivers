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

### Dynamic Allocation of Major Numbers
1. Some major device numbers are statically assigned to the most common devices. A list of those devices can be found in Documentation/devices.txt within the kernel source tree. 
2. Driver writer, you have a choice: you can simply pick a number that appears to be unused, or you can allocate major numbers in a dynamic manner. Picking a number may work as long as the only user of your driver is you; once your driver is more widely deployed, a randomly picked major number will lead to conflicts and trouble.
3. Thus, for new drivers, we strongly suggest that you use dynamic allocation to obtain your major device number, rather than choosing a number randomly from the ones that are currently free. In other words, your drivers should almost certainly be using
alloc_chrdev_region rather than register_chrdev_region.
4. The script to load a module that has been assigned a dynamic number can, therefore, be written using a tool such as awk to retrieve information from /proc/devices in order to create the files in /dev.
5. The following script, scull_load, is part of the scull distribution. The user of a driver that is distributed in the form of a module can invoke such a script from the system’s rc.local file or call it manually whenever the module is needed.
6. The script can be adapted for another driver by redefining the variables and adjusting the mknod lines. The script just shown creates four devices because four is the default in the scull sources.
7. The last few lines of the script may seem obscure: why change the group and mode of a device? The reason is that the script must be run by the superuser, so newly created special files are owned by root. The permission bits default so that only root has
write access, while anyone can get read access. 
8. A scull_unload script is also available to clean up the /dev directory and remove the
module.
9. As an alternative to using a pair of scripts for loading and unloading, you could write an init script, ready to be placed in the directory your distribution uses for these scripts. As part of the scull source, we offer a fairly complete and configurable example of an init script, called scull.init; it accepts the conventional arguments—start, stop, and restart—and performs the role of both scull_load and scull_unload.
10. The best way to assign major numbers, in our opinion, is by defaulting to dynamic allocation while leaving yourself the option of specifying the major number at load time, or even at compile time. The scull implementation works in this way; it uses a
global variable, scull_major, to hold the chosen number (there is also a scull_minor for the minor number). The variable is initialized to SCULL_MAJOR, defined in scull.h. The default value of SCULL_MAJOR in the distributed source is 0, which means “use
dynamic assignment.” The user can accept the default or choose a particular major number, either by modifying the macro before compiling or by specifying a value for scull_major on the insmod command line. Finally, by using the scull_load script, the
user can pass arguments to insmod on scull_load’s command line.‡Here’s the code we use in scull’s source to get a major number:
- **if (scull_major) {**
-  **dev = MKDEV(scull_major, scull_minor);**
 - **result = register_chrdev_region(dev, scull_nr_devs, "scull");**
- **} else {**
 - **result = alloc_chrdev_region(&dev, scull_minor, scull_nr_devs, "scull");
 - **scull_major = MAJOR(dev);**
- **}**
- **if (result < 0) {**
- **printk(KERN_WARNING "scull: can't get major %d\n", scull_major);**
 - **return result;**
- **}**

### Some Important Data Structures
1.  Most of the fundamental driver operations involve three important kernel data structures, called file_operations, file,
and inode. 
### File Operations
1. The file_operations structure is how a char driver sets up this connection. The structure, defined in <linux/fs.h>, is a collection of function pointers. Each open file (represented internally by a file structure, which we will examine shortly) is associated with its own set of functions (by including a field called f_op that points to a file_operations structure). The operations are mostly in charge of implementing the system calls and are therefore, named open, read, and so on.
2. We can consider the file to be an “object” and the functions operating on it to be its “methods,” using object-oriented programming
terminology to denote actions declared by an object to act on itself.
3. Conventionally, a file_operations structure or a pointer to one is called fops (or some variation thereof). Each field in the structure must point to the function in the driver that implements a specific operation, or be left NULL for unsupported operations. 
4. The scull device driver implements only the most important device methods. Its file_operations structure is initialized as follows:
- **struct file_operations scull_fops = {**
 - **.owner = THIS_MODULE,**
 - **.llseek = scull_llseek,**
 - **.read = scull_read,**
 - **.write = scull_write,**
 - **.ioctl = scull_ioctl,**
 - **.open = scull_open,**
 - **.release = scull_release,**
- **};**
This declaration uses the standard C tagged structure initialization syntax.
5. As you read through the list of file_operations methods, you will note that a number of parameters include the string \__user. This annotation is a form of documentation, noting that a pointer is a user-space address that cannot be directly dereferenced. For normal compilation, \__user has no effect, but it can be used by external checking software to find misuse of user-space addresses

### The file Structure
1. The file structure represents an open file. (It is not specific to device drivers; every open file in the system has an associated struct file in kernel space.) It is created by the kernel on open and is passed to any function that operates on the file, until
the last close. After all instances of the file are closed, the kernel releases the data structure.
2. In the kernel sources, a pointer to struct file is usually called either file or filp (“file pointer”). We’ll consistently call the pointer filp to prevent ambiguities with the structure itself. Thus, file refers to the structure and filp to a pointer to the
structure.
3. **void \*private_data;**
The open system call sets this pointer to NULL before calling the open method for the driver. You are free to make its own use of the field or to ignore it; you can use the field to point to allocated data, but then you must remember to free that memory in the release method before the file structure is destroyed by the kernel. private_data is a useful resource for preserving state information across system calls and is used by most of our sample modules.
## The inode Structure

1. The inode structure is used by the kernel internally to represent files. Therefore, it is different from the file structure that represents an open file descriptor. There can be numerous file structures representing multiple open descriptors on a single file, but
they all point to a single inode structure.
2. The inode structure contains a great deal of information about the file. As a general rule, only two fields of this structure are of interest for writing driver code:
**dev_t i_rdev;**
3. For inodes that represent device files, this field contains the actual device number.
**struct cdev *i_cdev;**
4. struct cdev is the kernel’s internal structure that represents char devices; this field contains a pointer to that structure when the inode refers to a char device file.
5. The kernel developers have added two macros that can be used to obtain the major and minor number from an inode:
**unsigned int iminor(struct inode \*inode);**
**unsigned int imajor(struct inode \*inode);**
6. In the interest of not being caught by the next change, these macros should be used instead of manipulating i_rdev directly.

### Char Device Registration

1. The kernel uses structures of type struct cdev to represent char devices internally. Before the kernel invokes your device’s operations, you must allocate and register one or more of these structures.* To do so, your code should include <linux/cdev.h>, where the structure and its associated helper functions are defined.
2. There are two ways of allocating and initializing one of these structures. If you wish to obtain a standalone cdev structure at runtime, you may do so with code such as:
**struct cdev *my_cdev = cdev_alloc( );**
**my_cdev->ops = &my_fops;**
3. you will want to embed the cdev structure within a device-specific structure of your own; that is what scull does. In that case, you should initialize the structure that you have already allocated with:
**void cdev_init(struct cdev *cdev, struct file_operations *fops);**
4. 


