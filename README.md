# Linux

Software developers use abstraction as a tool when building an operating system and its applications. There are many terms for an abstracted subdivision in computer software—including subsystem, module, and package—but we’ll use the term component 

A Linux system has three main levels. Figure 1-1 shows these levels and some of the components inside each level. The hardware is at the base. Hardware includes the memory as well as one or more central processing units (CPUs) to perform computation and to read from and write to memory. Devices such as disks and network interfaces are also part of the hardware.

The next level up is the kernel, which is the core of the operating system. 
The kernel is software residing in memory that tells the CPU where to look for its next task. Acting as a mediator, the kernel manages the hardware (especially main memory) and is the primary interface between the hardware and any running program.

Processes—the running programs that the kernel manages—collectively make up the system’s upper level, called user space. (A more specific term for process is user process, regardless of whether a user directly interacts with the process. For example, all web servers run as user processes.)

There is a critical difference between how the kernel and the user processes run: the kernel runs in kernel mode, and the user processes run in user mode. Code running in kernel mode has unrestricted access to the processor and main memory. This is a powerful but dangerous privilege that allows the kernel to easily corrupt and crash the entire system. The memory area that only the kernel can access is called kernel space.

User mode, in comparison, restricts access to a (usually quite small) subset of memory and safe CPU operations. User space refers to the parts of main memory that the user processes can access. If a process makes a mistake and crashes, the consequences are limited and can be cleaned up by the kernel. This means that if your web browser crashes, it probably won’t take down the scientific computation that has been running in the background for days.

The Linux kernel can run kernel threads, which look much like processes but have access to kernel space. Some examples are kthreadd and kblockd.

All input and output from peripheral devices flows through main memory, also as a bunch of bits. A CPU is just an operator on memory; it reads its instructions and data from the memory and writes data back out to the memory.

You’ll often hear the term state in reference to memory, processes, the kernel, and other parts of a computer system. Strictly speaking, a state is a particular arrangement of bits. For example, if you have four bits in your memory, 0110, 0001, and 1011 represent three different states.

When you consider that a single process can easily consist of millions of bits in memory, it’s often easier to use abstract terms when talking about states. Instead of describing a state using bits, you describe what something has done or is doing at the moment. For example, you might say, “The process is waiting for input” or, “The process is performing Stage 2 of its startup.”

NOTE

Because it’s common to refer to the state in abstract terms rather than to the actual bits, the term image refers to a particular physical arrangement of bits.
One of the kernel’s tasks is to split memory into many subdivisions, and it must maintain certain state information about those subdivisions at all times. Each process gets its own share of memory, and the kernel must ensure that each process keeps to its share.

The kernel is in charge of managing tasks in four general system areas:

Processes: The kernel is responsible for determining which processes are allowed to use the CPU.
Memory: The kernel needs to keep track of all memory—what is currently allocated to a particular process, what might be shared between processes, and what is free.
Device drivers: The kernel acts as an interface between hardware (such as a disk) and processes. It’s usually the kernel’s job to operate the hardware.
System calls and support: Processes normally use system calls to communicate with the kernel.


If you’re interested in the detailed workings of a kernel, two good textbooks are Operating System Concepts, 10th edition, by Abraham Silberschatz, Peter B. Galvin, and Greg Gagne (Wiley, 2018), and Modern Operating Systems, 4th edition, by Andrew S. Tanenbaum and Herbert Bos (Prentice Hall, 2014).

Process management describes the starting, pausing, resuming, scheduling, and terminating of processes
Consider a system with a one-core CPU. Many processes may be able to use the CPU, but only one process can actually use the CPU at any given time. In practice, each process uses the CPU for a small fraction of a second, then pauses; then another process uses the CPU for another small fraction of a second; then another process takes a turn, and so on. The act of one process giving up control of the CPU to another process is called a context switch.

Each piece of time—called a time slice—gives a process enough time for significant computation (and indeed, a process often finishes its current task during a single slice). However, because the slices are so small, humans can’t perceive them, and the system appears to be running multiple processes at the same time (a capability known as multitasking).

The kernel is responsible for context switching. To understand how this works, let’s think about a situation in which a process is running in user mode but its time slice is up. Here’s what happens:

The CPU (the actual hardware) interrupts the current process based on an internal timer, switches into kernel mode, and hands control back to the kernel.
The kernel records the current state of the CPU and memory, which will be essential to resuming the process that was just interrupted.
The kernel performs any tasks that might have come up during the preceding time slice (such as collecting data from input and output, or I/O, operations).
The kernel is now ready to let another process run. The kernel analyzes the list of processes that are ready to run and chooses one.
The kernel prepares the memory for this new process and then prepares the CPU.
The kernel tells the CPU how long the time slice for the new process will last.
The kernel switches the CPU into user mode and hands control of the CPU to the process.



important question of when the kernel runs. The answer is that it runs between process time slices during a context switch.

1.3.2 Memory Management

The kernel must manage memory during a context switch, which can be a complex job. The following conditions must hold:

The kernel must have its own private area in memory that user processes can’t access.
Each user process needs its own section of memory.
One user process may not access the private memory of another process.
User processes can share memory.
Some memory in user processes can be read-only.
The system can use more memory than is physically present by using disk space as auxiliary.

Page Table ≈ The Master Index Card Catalog that the Kernel sets up for the MMU
Page: A chunk of the Virtual Memory space (what the program sees).
Frame (or Page Frame): A chunk of the Physical Memory (the actual RAM hardware).

Virtual Page X is located in Physical Frame Y."

Device Drivers and Management
System Calls and Support

There are several other kinds of kernel features available to user processes. For example, system calls (or syscalls) perform specific tasks that a user process alone cannot do well or at all. For example, the acts of opening, reading, and writing files all involve system calls.

Two system calls, fork() and exec(), are important to understanding how processes start:

fork() When a process calls fork(), the kernel creates a nearly identical copy of the process.
exec() When a process calls exec(program), the kernel loads and starts program, replacing the current process.

remember that a system call is an interaction between a process and the kernel. 

Pseudodevices look like devices to user processes, but they’re implemented purely in software. This means they don’t technically need to be in the kernel, but they are usually there for practical reasons. For example, the kernel random number generator device (/dev/random) would be difficult to implement securely with a user process.

Technically, a user process that accesses a pseudodevice must use a system call to open the device, so processes can’t entirely avoid system calls.

1.4 User Space

As mentioned earlier, the main memory that the kernel allocates for user processes is called user space. Because a process is simply a state (or image) in memory, user space also refers to the memory for the entire collection of running processes. (You may also hear the more informal term userland used for user space; sometimes this also means the programs running in user space.)

Most of the real action on a Linux system happens in user space. Though all processes are essentially equal from the kernel’s point of view, they perform different tasks for users.

most applications and services write diagnostic messages known as logs. Most programs use the standard syslog service to write log messages, but some prefer to do all of the logging themselves.

1.5 Users

The Linux kernel supports the traditional concept of a Unix user. A user is an entity that can run processes and own files. A user is most often associated with a username; for example, a system could have a user named billyjoe. However, the kernel does not manage the usernames; instead, it identifies users by simple numeric identifiers called user IDs. (You’ll learn more about how usernames correspond to user IDs in Chapter 7.)

Users exist primarily to support permissions and boundaries. Every user-space process has a user owner, and processes are said to run as the owner. A user may terminate or modify the behavior of its own processes (within certain limits), but it cannot interfere with other users’ processes. In addition, users may own files and choose whether to share them with other users.

A Linux system normally has a number of users in addition to the ones that correspond to the real human beings who use the system. You’ll read about these in more detail in Chapter 3, but the most important user to know about is root. The root user is an exception to the preceding rules because root may terminate and alter another user’s processes and access any file on the local system. For this reason, root is known as the superuser. A person who can operate as root—that is, who has root access—is an administrator on a traditional Unix system.

Operating as root can be dangerous. It can be difficult to identify and correct mistakes because the system will let you do anything, even if it is harmful to the system. For this reason, system designers constantly try to make root access as unnecessary as possible—for example, by not requiring root access to switch between wireless networks on a notebook. In addition, as powerful as the root user is, it still runs in the operating system’s user mode, not kernel mode.

User processes make up the environment that you directly interact with; the kernel manages processes and hardware. Both the kernel and processes reside in memory.

2 BASIC COMMANDS AND DIRECTORY HIERARCHY

or more details about Unix for beginners than you’ll find here, consider reading The Linux Command Line, 2nd edition (No Starch Press, 2019), UNIX for the Impatient, 2nd edition (Addison-Wesley Professional, 1995), and Learning the UNIX Operating System, 5th edition (O’Reilly, 2001).

 A shell is a program that runs commands, like the ones that users enter into a terminal window. These commands can be other programs or built-in features of the shell. The shell also serves as a small programming environment. Unix programmers often break common tasks into smaller components and use the shell to manage tasks and piece things together.

shell scripts—text files that contain a sequence of shell commands. 

Linux uses an enhanced version of the Bourne shell called bash or the “Bourne-again” shell. The bash shell is the default shell on most Linux distributions, and /bin/sh is normally a link to bash on a Linux system. You should use the bash shell when running the examples in this book.

Unix processes use I/O streams to read and write data. Processes read data from input streams and write data to output streams. Streams are very flexible. For example, the source of an input stream can be a file, a device, a terminal window, or even the output stream from another process.

Learning the vi and Vim Editors: Unix Text Processing, 7th edition, by Arnold Robbins, Elbert Hannah, and Linda Lamb (O’Reilly, 2008), can tell you everything you need to know about vi. For Emacs, use the online tutorial: start Emacs, press CTRL-H, and then type T. Or read GNU Emacs Manual, 18th edition, by Richard M. Stallman (Free Software Foundation, 2018).

Don’t confuse error messages with warning messages. Warnings often look like errors, but they contain the word warning. A warning usually means something is wrong but the program will try to continue running anyway. To fix a problem noted in a warning message, you may have to hunt down a process and kill it before doing anything else. (You’ll learn about listing and killing processes in Section 2.16.)

PIDs are unique for each process running on a system. However, after a process terminates, the kernel can eventually reuse the PID for a new process.
















