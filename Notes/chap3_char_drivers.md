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
