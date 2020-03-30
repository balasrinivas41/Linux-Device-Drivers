# Linux-Device-Drivers

#Preparing the system to run the code:

sudo apt-get install build-essential linux-headers-$(uname -r)



#Makefile to compile the source code:

obj-m = hello.o
all:
        make -C /lib/modules/$(shell uname -r)/build/ M=$(PWD) modules
clean:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean





#Compiling and loading the module:

make



#Now we will use insmod to load the hello.ko object:

sudo insmod hello.ko


#To see the message, we need to read the kern.log in /var/log directory:

tail /var/log/kern.log

