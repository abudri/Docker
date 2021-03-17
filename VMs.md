**Below Taken from [A comprehensive introduction to Docker, Virtual Machines, and Containers
](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/) by shota jolbordi**

## Table of Contents
* [Computer Basics](https://github.com/abudri/Docker/blob/main/VMs.md#computer-basics)
* [Hypervisors](https://github.com/abudri/Docker/blob/main/VMs.md#hypervisors)
* [Containers](https://github.com/abudri/Docker/blob/main/VMs.md#containers)

## Computer Basics

Every computer, ever, be it the gigantic web server running Linux or your overpriced iPhone X, has 4 essential physical components:

* Processor (CPU),
* Memory (RAM),
* Storage (HDD / SSD),
* The network card (NIC).

The main task of any operating system is to basically manage those 4 resources. The part of the operating system that does this is called the **Kernel**, also referred to as the **Core** (Me: Also, the "heart" of the OS).

The kernel, simply put, is a part of the OS that controls the hardware. The kernel controls drivers for different IO devices such as a mouse, keyboard, headphones, microphone…etc. The kernel is the first program loaded when the computer is turned on, right after the bootloader, and then it handles the rest of the startup process. An absolute majority of the time that it takes to turn on the computer, **is because of the Kernel**.

![image](https://user-images.githubusercontent.com/17362519/111316969-172d4500-863a-11eb-98c0-7767a7f06d10.png)

Each operating systems has its own implementation of the kernel, but in fact they do all the same thing: they control the hardware.

So how is it possible to run one OS inside another? Essentially what we need is a program that enables the **Guest OS** (the operating system that is running inside another operating system) to control the hardware of the **Host OS** (an operating system that has a guest OS running inside of it).

## Hypervisors

The hypervisor, also referred to as **Virtual Machine Manager (VMM)**, is what enables virtualization (running several operating systems on one physical computer). It allows the host computer to share its resources between VMs.

There are 2 types of Hypervisors:

**Type 1, also called “Bare Metal Hypervisor”**

This software is installed right on top of the underlying machine’s hardware (so, in this case, there is no Host OS, there are only Guest OS’s). You would do this on a machine on which the whole purpose was to run many virtual machines.

Type 1 hypervisors have their own device drivers and interact with hardware directly unlike type 2 hypervisors. That’s what makes them faster, simpler and hence more stable.

**Type 2, also called “Hosted Hypervisor”**

This is a program that is installed on top of the operating system. You are probably more familiar with it, like VirtualBox or VMware Workstation. This type of hypervisor is something like a “translator” that translates the guest operating system’s system calls into the host operating system’s system calls.

The **system calls (syscalls)** are a way in which a program requests a service from a Kernel, and the Kernel does — remember what? It manages underlying hardware.

For example, in your program, say you want to copy the content of one file into another. Pretty straightforward right? For this, you need to take some bytes from one part of your Hard Disk and put them into another part. So basically, you are doing stuff with a physical resource, the Hard Disk in this example, and you would need to initiate a **system call** to do this. Of course in all programming languages, this is abstracted away from you, but you get the point.

Since all OS Kernels, despite being implemented in different ways, do the same job (control hardware), we just need a program that will “translate” a guest OS’s system calls to control the hardware.

An upside of a Type 2 hypervisor is that in this case we don’t have to worry about underlying hardware and it’s drivers. We really just need to delegate the job to the host OS, which will manage this stuff for us. The downside is that it creates a resource overhead, and multiple layers sitting on top of each other make things complicated and lowers the performance.

![image](https://user-images.githubusercontent.com/17362519/111483715-6e4f1a80-870b-11eb-977b-ab4bd3eca888.png)

## Containers

Virtual machines are not the only virtualization technique. In the case of a virtual machine, we have a full-blown virtual computer, in its entirety, with its own dedicated Kernel. We allocate RAM for it, we allocate memory for it, and we interact with it as if it was a standalone computer.

There are several problems with this. First and most obvious is inefficient resource management. Once you allocate some resources for a VM, it’s going to hold onto them as long as it’s running.

For example: if you allocate 4 GB of RAM and 40GB of disk memory for a VM, once you run it, those resources will be unavailable as long as this VM is running. It might only need 1 GB of RAM at some moment, and you might be lacking RAM for some other process in another VM or host machine. But since it has this amount of RAM allocated, it’s just going to sit there unused.

Another problem is boot up time. Since the VM has its own Kernel, in case you need to restart your machine, it will need to boot up an entire Kernel. While the machine is rebooting, your service that was running in VM will be unavailable.

### Containers to the rescue

To put it simply, a container is a virtual machine without a Kernel. Instead, it is using the Kernel of a host operating system. To make this possible, we need a set of software and libraries that will allow containers to use the underlying OS Kernel, and sort of “link” them if you wish. Such libraries are, for example, **“liblxc”** and **“libcontainer”** (this last one is developed by Docker, Inc and is used inside docker engine).

Containers have their own allocated filesystem and IP. Libraries, binaries, services are installed inside a container, however, all the system calls and Kernel functionality comes from the underlying host OS.

Containers are very lightweight. Boot up and restart happens very fast because they don’t need to start up the Kernel every time. They don’t waste physical resources since they don’t need them to be allocated for their Kernel, as they don’t have a separate Kernel.

One drawback is that _**it’s only possible to run containers of the same type as the underlying OS**_. You can’t run Linux containers on Windows or Mac, because they need Linux Kennel to operate. The solution for Mac and Windows users would be to install a type 2 hypervisor such as **VirtualBox** or **WMware Workstation**, boot up the Linux machine, and then run Linux containers inside of it (in fact that’s what Docker for Mac and Docker for Windows do, but they use native hypervisors that come with the respective OS).

Setting up and running Linux containers is not that straightforward. It’s troublesome and requires a decent knowledge of Linux. Managing them is even more tedious.

As I’ve mentioned above, what Docker, Inc does is it makes Linux containers easy to use and available to everybody, and you do not have to be a Linux geek to use Linux containers nowadays thanks to docker.

![image](https://user-images.githubusercontent.com/17362519/111528124-234afc80-8737-11eb-91dd-c6a9e70b4402.png)

### Containers VS Virtual Machines

From the previous section about containers, you might think that containers are just better virtualization solutions than VM’s, but that’s not how it is.

A container’s purpose is running processes in an isolated environment, for docker each container for every single process. VM’s are for emulating an entire machine. Nowadays only Linux and windows containers exist, but there are all kinds of hypervisors to emulate any kind of operating system. You can run windows 10 inside an iPad if you wish. Those 2 are different technologies and they don’t compete with each other.

VM’s are more secure, since containers make system calls directly to the Kernel. This opens up a whole verity of vulnerabilities.

Some low-level software that messes with a Kernel directly should be sandboxed inside a virtual machine.

Often you can see docker containers running inside virtual machines in the production environment, so VM’s and containers actually stick together very well.

### Docker Images and Containers
Docker introduces several concepts that simplify…or I would rather say revolutionize the usage of Linux Containers

Linux containers in docker are made from templates called “images”. An image is a basically a binary file that holds the state of a Linux machine (without the Kernel of course). You can draw a parallel to VM’s disk images such as .vdi, .vmdk or .vhd files.

Docker’s approach to images is different from a VM’s. In a VM you would just mount a disk image, run the VM, and you would have a running instance of a machine. Whenever you modify filesystem in VM, install or remove anything, all of this is reflected on an image you’ve mounted. The image is basically the Hard Disk of the machine.

In docker, images are read-only — you don’t run images directly, instead, you make a copy of an image and run it. This running instance of an image is called a container. By doing this you can have several instances of the same Linux container running at the same time, made from the same template, that are images. Whatever happens with a container does not affect the image it was made from. You can make as many instances of a container from an image as your hardware allows you to run.







