---
layout:     post
title:      Unikernels\: Rise of the virtual library operating system
subtitle:   A Paper Digest
date:       2019-10-28
author:     Susu
header-img: img/posts20191028/header_bg.jpg
catalog: 	 true
tags:
    - Virtualization
    - Library OS
    - Mirage OS
    - Unikernel
---

This post is a summary of paper "Madhavapeddy, Anil, and David J. Scott. "Unikernels: Rise of the virtual library operating system." _Queue_ 11, no. 11 (2013): 30.".

This paper is about Mirage OS, one instance of Library OS in the virtualization world.

## Motivations:
There are mainly three motivations for Mirage OS:
* Currently, in traditional operating system, the software stack has a lot of layers, for instance, libraries necessary for application, TCP/IP stack, file system interface, User-space threads and processes,   And all of these layers sit underneath the application code.  So the author thought, is there any way to reduce the number of layers under the application code?
* The transition to single purpose appliances.  In current cloud, usually virtual machines are running a standard operating systems.  Even though such full-fledge system can support many different kind of applications, in most cases virtual machines are launched to work as database or server.  So, this bring the author an opportunity to do optimization.  Firstly, as now each VM only runs one application, the performance of the application can be improved by adapting the setting of VM to its application.  Secondly, if we remove the redundant functionality of standard OS, making our system more light-weight, the attack surfaces would become small, thereby improving security.
* The limitation of traditional operating system in virtual world.  First of all, current hypervisor is designed to support dynamic scale both by adding more memory and CPU cores to existing machine and by spawning new virtual machines.  But current system have been designed before such technique came about, like some systems do not support memory hot plug-in.  Secondly, usually an external application-level load balancer is utilized, which spawns more VMs while load spikes. However, the traditional OS is not optimized both for boot time and size.  Therefore, some extra VMs are launched in case of such unpredictable load increases.  But in most of the time, these VMs are idling.  This incurs a wastes of money and computing resources.  Lastly, currently we do not necessary need the function of multiplexing hardware provided by standard OS, because in virtualization world, hardware multiplex is done by hypervisor.   But noted that even though hypervisor provides hardware multiplex, but this is not enough.  We can not just remove the layer of guest operating system which traditionally resides between hypervisor and running applications, as OS also provides some other services necessary for applications, like TCP/IP stacks, and file system interfaces.

## The idea of Library OS
In a nutshell, Library OS is about encapsulating application-specific services which usually provided by OS together, and runs in the applications' address space.  Such application-specific services are not hardware support services, like drivers, file system interfaces.  An example of application-specific services is graphic user interface.  

So using library OS, it is a must to implement such hardware support services.  And noting that now applications are sharing a set of library OS, so a set of policies are necessary to guarantee application isolation.

In the world of Library OS, there is no user mode and kernel mode.  Application can directly access hardware without the transition from user mode to kernel mode.  And there will be no data transfer from user space to kernel space, which improve application performance.

* Isolating applications using the same library OS is very complex.
* The implementation of hardware drivers. As we all know, there are tons of different hardware, each of them requires a specific driver.  Writing that many drivers is certainly a tedious work.

## Unikernel: Library OS in vritualization world

Fortunately, we are introduced to a brand new world by the virtualization techniques.  Hypervisor provides VMs with CPU time and strongly isolated virtual devices for networking etc, which perfectly solve the aforementioned two drawbacks.
* Isolation can be achieved by launching new VMs.  Each VM  runs only an application, which prevents one application from being messed up with another application.
* Hypervisor will take care of writing drivers for various hardware. And only drivers for virtual hardware are needed on Library OS side.

Unikernel is a system adapting the idea of Library OS.  An application combined with Library OS along with other necessary services forms a Unikernel.  In this paper, the author designed an instance of Unikernel called Mirage OS. 

So, let’s talking about these “other necessary services” now.  These services refer to the necessary services neither provided by hypervisor nor Library OS.  For example, the aforementioned TCP/IP stacks, virtual hardware drivers.  

In traditional OS, these services are usually realized by C (yes, the programming language C), as C excels at low-level programs such as device drivers.   However, C lacks the abstraction facilities of high-level languages and demands careful manual tracking of resources such as memory buffers.  So they decided to use Ocaml

## Implementation Details
I don’t bother to mention these details.
But there are few things I find interesting:
* The output of the compiler would be a Unikernel instead of just an executable file of application.  This unikernel includes application libraries, and all other services required by this application.  And since this Unikernel is supposed to be able to run directly on a VM, so it also has boot support and garbage collector.
* Because Ocaml has the attribute of metaprogramming.  So the compiler can optimize the entire Unikernel as a whole.  Such optimization also brings downside.  Because of the optimization, the output binary of complier lacks some blocks of functionality, so any change of the configuration file would result in compiling the entire Unikernel again. 
* Also, benefit from the idea of “module” in Ocaml, the Unikernel can be developed at our own computer and then combined with MirageOS shared on GitHub.
* An Unikernel runs only in one virtual address space, the entire Unikernel runs as a process on Hypervisor.  Once this application is done with its job, this VM is terminated.

## Benefits
Unikernel is light-weight compared with the traditional VM running an application.  So live migration is easy to achieve.  Also, the boot time is very small.  Just a snap of finger, or a blink of eye, many VMs can be launched and ready to serve.
