**Below Taken from [A comprehensive introduction to Docker, Virtual Machines, and Containers
](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/) by shota jolbordi**

## Table of Contents
* [Computer Basics](https://github.com/abudri/Docker/new/main#computer-basics)
* [Hypervisors](https://github.com/abudri/Docker/blob/main/VMs.md#hypervisors)

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

The system calls (syscalls) are a way in which a program requests a service from a Kernel, and the Kernel does — remember what? It manages underlying hardware.

For example, in your program, say you want to copy the content of one file into another. Pretty straightforward right? For this, you need to take some bytes from one part of your Hard Disk and put them into another part. So basically, you are doing stuff with a physical resource, the Hard Disk in this example, and you would need to initiate a system call to do this. Of course in all programming languages, this is abstracted away from you, but you get the point.

Since all OS Kernels, despite being implemented in different ways, do the same job (control hardware), we just need a program that will “translate” a guest OS’s system calls to control the hardware.

An upside of a Type 2 hypervisor is that in this case we don’t have to worry about underlying hardware and it’s drivers. We really just need to delegate the job to the host OS, which will manage this stuff for us. The downside is that it creates a resource overhead, and multiple layers sitting on top of each other make things complicated and lowers the performance.

![image](https://user-images.githubusercontent.com/17362519/111483715-6e4f1a80-870b-11eb-977b-ab4bd3eca888.png)


