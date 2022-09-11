# **1.5 OPERATING SYSTEM CONCEPTS**

Most operating systems provide certain basic concepts and abstractions such as
processes, address spaces, and files that are central to understanding them. In the
following sections, we will look at some of these basic concepts ever so briefly, as 

<center>OPERATING SYSTEM CONCEPTS</center>


an introduction. We will come back to each of them in great detail later in this
book. To illustrate these concepts we will, from time to time, use examples, generally drawn from UNIX. Similar examples typically exist in other systems as well,
however, and we will study some of them later.

### **1.5.1 Proceses**
A key concept in all operating systems is the **process.** A process is basically a
program in execution. Associated with each process is its **address space**, a list of
memory locations from 0 to some maximum, which the process can read and write.
The address space contains the executable program, the program’s data, and its
stack. Also associated with each process is a set of resources, commonly including
registers (including the program counter and stack pointer), a list of open files, outstanding alarms, lists of related processes, and all the other information needed to
run the program. A process is fundamentally a container that holds all the information needed to run a program.

We will come back to the process concept in much more detail in Chap. 2. For
the time being, the easiest way to get a good intuitive feel for a process is to think
about a multiprogramming system. The user may have started a video editing program and instructed it to convert a one-hour video to a certain format (something
that can take hours) and then gone off to surf the Web. Meanwhile, a background
process that wakes up periodically to check for incoming email may have started
running. Thus we have (at least) three active processes: the video editor, the Web
browser, and the email receiver. Periodically, the operating system decides to stop
running one process and start running another, perhaps because the first one has
used up more than its share of CPU time in the past second or two.

When a process is suspended temporarily like this, it must later be restarted in
exactly the same state it had when it was stopped. This means that all information
about the process must be explicitly saved somewhere during the suspension. For
example, the process may have sev eral files open for reading at once. Associated
with each of these files is a pointer giving the current position (i.e., the number of
the byte or record to be read next). When a process is temporarily suspended, all
these pointers must be saved so that a read call executed after the process is restarted will read the proper data. In many operating systems, all the information about
each process, other than the contents of its own address space, is stored in an operating system table called the **process table**, which is an array of structures, one for
each process currently in existence.
Thus, a (suspended) process consists of its address space, usually called the
core image (in honor of the magnetic core memories used in days of yore), and its
process table entry, which contains the contents of its registers and many other
items needed to restart the process later.

The key process-management system calls are those dealing with the creation and termination of processes. Consider a typical example. A process called the
**command interpreter** or shell reads commands from a terminal. The user has just 

<center>INTRODUCTION</center>

typed a command requesting that a program be compiled. The shell must now create a new process that will run the compiler. When that process has finished the
compilation, it executes a system call to terminate itself.

If a process can create one or more other processes (referred to as **child processes**) and these processes in turn can create child processes, we quickly arrive at
the process tree structure of Fig. 1-13. Related processes that are cooperating to
get some job done often need to communicate with one another and synchronize
their activities. This communication is called **interprocess communication**, and
will be addressed in detail in Chap. 2.

![](113.jpeg)

<center>  Figure 1-13. A process tree. Process A created two child processes, B and C.
Process B created three child processes, D, E, and F</center>

Other process system calls are available to request more memory (or release
unused memory), wait for a child process to terminate, and overlay its program
with a different one. 

Occasionally, there is a need to convey information to a running process that is
not sitting around waiting for this information. For example, a process that is communicating with another process on a different computer does so by sending messages to the remote process over a computer network. To guard against the possibility that a message or its reply is lost, the sender may request that its own operating system notify it after a specified number of seconds, so that it can retransmit
the message if no acknowledgement has been received yet. 

After setting this timer, the program may continue doing other work.
When the specified number of seconds has elapsed, the operating system sends
an **alarm signal** to the process. The signal causes the process to temporarily suspend whatever it was doing, save its registers on the stack, and start running a special signal-handling procedure, for example, to retransmit a presumably lost message. When the signal handler is done, the running process is restarted in the state it was in just before the signal. Signals are the software analog of hardware interrupts and can be generated by a variety of causes in addition to timers expiring.
Many traps detected by hardware, such as executing an illegal instruction or using
an invalid address, are also converted into signals to the guilty process.

Each person authorized to use a system is assigned a **UID (User IDentification)** by the system administrator. Every process started has the UID of the person who started it. A child process has the same UID as its parent. Users can be members of groups, each of which has a **GID (Group IDentification)**.

<center>OPERATING SYSTEM CONCEPTS</center>

One UID, called the **superuser** (in UNIX), or **Administrator** (in Windows),
has special power and may override many of the protection rules. In large installations, only the system administrator knows the password needed to becomesuperuser, but many of the ordinary users (especially students) devote considerable
effort seeking flaws in the system that allow them to become superuser without the password.
We will study processes and interprocess communication in Chap. 2.

### **Address Spaces**
Every computer has some main memory that it uses to hold executing programs. In a very simple operating system, only one program at a time is in memory. To run a second program, the first one has to be removed and the second one
placed in memory.

More sophisticated operating systems allow multiple programs to be in memory at the same time. To keep them from interfering with one another (and with the operating system), some kind of protection mechanism is needed. While this mechanism has to be in the hardware, it is controlled by the operating system. 

The above viewpoint is concerned with managing and protecting the computer’s main memory. A different, but equally important, memory-related issue is managing the address space of the processes. Normally, each process has some set of addresses it can use, typically running from 0 up to some maximum. In the simplest case, the maximum amount of address space a process has is less than the main memory. In this way, a process can fill up its address space and there will be enough room in main memory to hold it all.

However, on many computers addresses are 32 or 64 bits, giving an address space of 232 or 264 bytes, respectively. What happens if a process has more address space than the computer has main memory and the process wants to use it all? In the first computers, such a process was just out of luck. Nowadays, a technique called virtual memory exists, as mentioned earlier, in which the operating system keeps part of the address space in main memory and part on disk and shuttles pieces back and forth between them as needed. In essence, the operating system creates the abstraction of an address space as the set of addresses a process may reference. The address space is decoupled from the machine’s physical memory and may be either larger or smaller than the physical memory. Management of address spaces and physical memory form an important part of what an operating system does, so all of Chap. 3 is devoted to this topic.

### **1.5.3 Files**

Another key concept supported by virtually all operating systems is the file
system. As noted before, a major function of the operating system is to hide the
peculiarities of the disks and other I/O devices and present the programmer with a 

<center>INTRODUCTION</center>
nice, clean abstract model of device-independent files. System calls are obviously
needed to create files, remove files, read files, and write files. Before a file can be
read, it must be located on the disk and opened, and after being read it should be
closed, so calls are provided to do these things.

To provide a place to keep files, most PC operating systems have the concept
of a directory as a way of grouping files together. A student, for example, might
have one directory for each course he is taking (for the programs needed for that
course), another directory for his electronic mail, and still another directory for his
World Wide Web home page. System calls are then needed to create and remove
directories. Calls are also provided to put an existing file in a directory and to remove a file from a directory. Directory entries may be either files or other directories. This model also gives rise to a hierarchy—the file system—as shown in
Fig. 1-14.

![](114.jpeg)

The process and file hierarchies both are organized as trees, but the similarity stops there. Process hierarchies usually are not very deep (more than three levels is unusual), whereas file hierarchies are commonly four, fiv e, or even more levels deep. Process hierarchies are typically short-lived, generally minutes at most, whereas the directory hierarchy may exist for years. Ownership and protection also differ for processes and files. Typically, only a parent process may control or even 

<center>OPERATING SYSTEM CONCEPTS</center>

access a child process, but mechanisms nearly always exist to allow files and directories to be read by a wider group than just the owner.

Every file within the directory hierarchy can be specified by giving its **path name** from the top of the directory hierarchy, the root directory. Such absolute
path names consist of the list of directories that must be traversed from the **root directory** to get to the file, with slashes separating the components. In Fig. 1-14, the
path for file *CS101 is /Faculty/Prof.Brown/Courses/CS101.* The leading slash indicates that the path is absolute, that is, starting at the root directory. As an aside, in
Windows, the backslash ( / ) character is used as the separator instead of the slash (/)
character (for historical reasons), so the file path given above would be written as
*\Faculty\Prof.Brown\Courses\CS101*. Throughout this book we will generally use the UNIX convention for paths.

At every instant, each process has a current **working directory**, in which path
names not beginning with a slash are looked for. For example, in Fig. 1-14, if
*/Faculty/Prof.Brown* were the working directory, use of the path *Courses/CS101* would yield the same file as the absolute path name given above. Processes can change their working directory by issuing a system call specifying the new working directory.

Before a file can be read or written, it must be opened, at which time the permissions are checked. If the access is permitted, the system returns a small integer
called a **file descriptor** to use in subsequent operations. If the access is prohibited,
an error code is returned.

Another important concept in UNIX is the mounted file system. Most desktop
computers have one or more optical drives into which CD-ROMs, DVDs, and Bluray discs can be inserted. They almost always have USB ports, into which USB
memory sticks (really, solid state disk drives) can be plugged, and some computers
have floppy disks or external hard disks. To provide an elegant way to deal with
these removable media UNIX allows the file system on the optical disc to be attached to the main tree. Consider the situation of Fig. 1-15(a). Before the mount
call, the **root file system**, on the hard disk, and a second file system, on a CDROM, are separate and unrelated.

However, the file system on the CD-ROM cannot be used, because there is no way to specify path names on it. UNIX does not allow path names to be prefixed by a drive name or number; that would be precisely the kind of device dependence that operating systems ought to eliminate. Instead, the mount system call allows
the file system on the CD-ROM to be attached to the root file system wherever the program wants it to be. In Fig. 1-15(b) the file system on the CD-ROM has been
mounted on directory b, thus allowing access to files */b/x and /b/y*. If directory b
had contained any files they would not be accessible while the CD-ROM was
mounted, since /b would refer to the root directory of the CD-ROM. (Not being
able to access these files is not as serious as it at first seems: file systems are nearly
always mounted on empty directories.) If a system contains multiple hard disks,
they can all be mounted into a single tree as well.

<center>INTRODUCTION</center>

![](115.jpeg)

Another important concept in UNIX is the **special file.** Special files are provided in order to make I/O devices look like files. That way, they can be read and
written using the same system calls as are used for reading and writing files. Two
kinds of special files exist: **block special files** and **character special files**. Block
special files are used to model devices that consist of a collection of randomly addressable blocks, such as disks. By opening a block special file and reading, say,
block 4, a program can directly access the fourth block on the device, without
regard to the structure of the file system contained on it. Similarly, character special files are used to model printers, modems, and other devices that accept or output a character stream. By convention, the special files are kept in the /dev directory. For example, */dev*/lp might be the printer (once called the line printer).

The last feature we will discuss in this overview relates to both processes and files: pipes. A **pipe** is a sort of pseudofile that can be used to connect two processes, as shown in Fig. 1-16. If processes A and B wish to talk using a pipe, they
must set it up in advance. When process A wants to send data to process B, it writes on the pipe as though it were an output file. In fact, the implementation of a pipe is
very much like that of a file. Process B can read the data by reading from the pipe as though it were an input file. Thus, communication between processes in UNIX looks very much like ordinary file reads and writes. Stronger yet, the only way a process can discover that the output file it is writing on is not really a file, but a
pipe, is by making a special system call. File systems are very important. We will
have much more to say about them in Chap. 4 and also in Chaps. 10 and 11.

![](116.jpeg)

<center>OPERATING SYSTEM CONCEPTS</center>

### **1.5.4 Input/Ouput**

All computers have physical devices for acquiring input and producing output.
After all, what good would a computer be if the users could not tell it what to do
and could not get the results after it did the work requested? Many kinds of input
and output devices exist, including keyboards, monitors, printers, and so on. It is
up to the operating system to manage these devices.

Consequently, every operating system has an I/O subsystem for managing its
I/O devices. Some of the I/O software is device independent, that is, applies to many or all I/O devices equally well. Other parts of it, such as device drivers, are specific to particular I/O devices. In Chap. 5 we will have a look at I/O software.

### **1.5.5 Protection**

Computers contain large amounts of information that users often want to protect and keep confidential. This information may include email, business plans, tax returns, and much more. It is up to the operating system to manage the system security so that files, for example, are accessible only to authorized users.

As a simple example, just to get an idea of how security can work, consider
UNIX. Files in UNIX are protected by assigning each one a 9-bit binary protection code. The protection code consists of three 3-bit fields, one for the owner, one for other members of the owner’s group (users are divided into groups by the system administrator), and one for everyone else. Each field has a bit for read access,
a bit for write access, and a bit for execute access. These 3 bits are known as the
**rwx bits**. For example, the protection code *rwxr-x--x* means that the owner can
**r**ead, **w**rite, or e**x**ecute the file, other group members can read or execute (but not
write) the file, and everyone else can execute (but not read or write) the file. For a
directory, x indicates search permission. A dash means that the corresponding permission is absent.

In addition to file protection, there are many other security issues. Protecting
the system from unwanted intruders, both human and nonhuman (e.g., viruses) is
one of them. We will look at various security issues in Chap. 9.

### **1.5.6 The Shell**

The operating system is the code that carries out the system calls. Editors,
compilers, assemblers, linkers, utility programs, and command interpreters definitely are not part of the operating system, even though they are important and useful. At the risk of confusing things somewhat, in this section we will look briefly
at the UNIX command interpreter, the shell. Although it is not part of the operating system, it makes heavy use of many operating system features and thus serves
as a good example of how the system calls are used. It is also the main interface 

<center>INTRODUCTION</center>

between a user sitting at his terminal and the operating system, unless the user is
using a graphical user interface. Many shells exist, including sh, csh, ksh, and bash.
All of them support the functionality described below, which derives from the original shell (sh).

When any user logs in, a shell is started up. The shell has the terminal as standard input and standard output. It starts out by typing the **prompt**, a character
such as a dollar sign, which tells the user that the shell is waiting to accept a command. If the user now types

date

for example, the shell creates a child process and runs the *date* program as the
child. While the child process is running, the shell waits for it to terminate. When
the child finishes, the shell types the prompt again and tries to read the next input
line.

The user can specify that standard output be redirected to a file, for example, date >file

Similarly, standard input can be redirected, as in

sort <file1 >file2

which invokes the sort program with input taken from *file1* and output sent to *file2*.
The output of one program can be used as the input for another program by
connecting them with a pipe. Thus

cat file1 file2 file3 | sort >/dev/lp

invokes the *cat* program to concatenate three files and send the output to *sort* to
arrange all the lines in alphabetical order. The output of *sort* is redirected to the file
*/dev/lp*, typically the printer.
If a user puts an ampersand after a command, the shell does not wait for it to
complete. Instead it just gives a prompt immediately. Consequently,

cat file1 file2 file3 | sort >/dev/lp &

starts up the sort as a background job, allowing the user to continue working normally while the sort is going on. The shell has a number of other interesting features, which we do not have space to discuss here. Most books on UNIX discuss
the shell at some length (e.g., Kernighan and Pike, 1984; Quigley, 2004; Robbins,
2005).

Most personal computers these days use a GUI. In fact, the GUI is just a program running on top of the operating system, like a shell. In Linux systems, this
fact is made obvious because the user has a choice of (at least) two GUIs: Gnome
and KDE or none at all (using a terminal window on X11). In Windows, it is also
possible to replace the standard GUI desktop (Windows Explorer) with a different
program by changing some values in the registry, although few people do this.

<center>OPERATING SYSTEM CONCEPTS</center>

### **1.5.7 Ontogeny Recapitulates Phylogeny**

After Charles Darwin’s book *On the Origin of the Species* was published, the
German zoologist Ernst Haeckel stated that ‘‘ontogeny recapitulates phylogeny.’’
By this he meant that the development of an embryo (ontogeny) repeats (i.e., recapitulates) the evolution of the species (phylogeny). In other words, after fertilization, a human egg goes through stages of being a fish, a pig, and so on before turning into a human baby. Modern biologists regard this as a gross simplification, but it still has a kernel of truth in it.

Something vaguely analogous has happened in the computer industry. Each
new species (mainframe, minicomputer, personal computer, handheld, embedded
computer, smart card, etc.) seems to go through the development that its ancestors
did, both in hardware and in software. We often forget that much of what happens
in the computer business and a lot of other fields is technology driven. The reason
the ancient Romans lacked cars is not that they liked walking so much. It is because they did not know how to build cars. Personal computers exist *not* because
millions of people have a centuries-old pent-up desire to own a computer, but because it is now possible to manufacture them cheaply. We often forget how much
technology affects our view of systems and it is worth reflecting on this point from
time to time.

In particular, it frequently happens that a change in technology renders some
idea obsolete and it quickly vanishes. However, another change in technology
could revive it again. This is especially true when the change has to do with the
relative performance of different parts of the system. For instance, when CPUs
became much faster than memories, caches became important to speed up the
‘‘slow’’ memory. If new memory technology someday makes memories much
faster than CPUs, caches will vanish. And if a new CPU technology makes them
faster than memories again, caches will reappear. In biology, extinction is forever,
but in computer science, it is sometimes only for a few years.

As a consequence of this impermanence, in this book we will from time to
time look at ‘‘obsolete’’ concepts, that is, ideas that are not optimal with current
technology. Howev er, changes in the technology may bring back some of the
so-called ‘‘obsolete concepts.’’ For this reason, it is important to understand why a
concept is obsolete and what changes in the environment might bring it back again.

To make this point clearer, let us consider a simple example. Early computers
had hardwired instruction sets. The instructions were executed directly by hardware and could not be changed. Then came microprogramming (first introduced on
a large scale with the IBM 360), in which an underlying interpreter carried out the
‘‘hardware instructions’’ in software. Hardwired execution became obsolete. It
was not flexible enough. Then RISC computers were invented, and microprogramming (i.e., interpreted execution) became obsolete because direct execution
was faster. Now we are seeing the resurgence of interpretation in the form of Java
applets that are sent over the Internet and interpreted upon arrival. Execution speed 

<center>INTRODUCTION</center>

is not always crucial because network delays are so great that they tend to dominate. Thus the pendulum has already swung several cycles between direct execution and interpretation and may yet swing again in the future.


### **Large Memories**

Let us now examine some historical developments in hardware and how they
have affected software repeatedly. The first mainframes had limited memory. A
fully loaded IBM 7090 or 7094, which played king of the mountain from late 1959
until 1964, had just over 128 KB of memory. It was mostly programmed in assembly language and its operating system was written in assembly language to save
precious memory.

As time went on, compilers for languages like FORTRAN and COBOL got
good enough that assembly language was pronounced dead. But when the first
commercial minicomputer (the PDP-1) was released, it had only 4096 18-bit words
of memory, and assembly language made a surprise comeback. Eventually, minicomputers acquired more memory and high-level languages became prevalent on
them.

When microcomputers hit in the early 1980s, the first ones had 4-KB memories and assembly-language programming rose from the dead. Embedded computers often used the same CPU chips as the microcomputers (8080s, Z80s, and
later 8086s) and were also programmed in assembler initially. Now their descendants, the personal computers, have lots of memory and are programmed in C,
C++, Java, and other high-level languages. Smart cards are undergoing a similar
development, although beyond a certain size, the smart cards often have a Java
interpreter and execute Java programs interpretively, rather than having Java being
compiled to the smart card’s machine language.

### **Protection Hardware**

Early mainframes, like the IBM 7090/7094, had no protection hardware, so
they just ran one program at a time. A buggy program could wipe out the operating system and easily crash the machine. With the introduction of the IBM 360, a
primitive form of hardware protection became available. These machines could
then hold several programs in memory at the same time and let them take turns
running (multiprogramming). Monoprogramming was declared obsolete.

At least until the first minicomputer showed up—without protection hardware—so multiprogramming was not possible. Although the PDP-1 and PDP-8
had no protection hardware, eventually the PDP-11 did, and this feature led to multiprogramming and eventually to UNIX.

When the first microcomputers were built, they used the Intel 8080 CPU chip,
which had no hardware protection, so we were back to monoprogramming—one
program in memory at a time. It was not until the Intel 80286 chip that protection 

<center>OPERATING SYSTEM CONCEPTS</center>

hardware was added and multiprogramming became possible. Until this day, many
embedded systems have no protection hardware and run just a single program.

Now let us look at operating systems. The first mainframes initially had no
protection hardware and no support for multiprogramming, so they ran simple operating systems that handled one manually loaded program at a time. Later they acquired the hardware and operating system support to handle multiple programs at
once, and then full timesharing capabilities.

When minicomputers first appeared, they also had no protection hardware and
ran one manually loaded program at a time, even though multiprogramming was
well established in the mainframe world by then. Gradually, they acquired protection hardware and the ability to run two or more programs at once. The first
microcomputers were also capable of running only one program at a time, but later
acquired the ability to multiprogram. Handheld computers and smart cards went
the same route.

In all cases, the software development was dictated by technology. The first
microcomputers, for example, had something like 4 KB of memory and no protection hardware. High-level languages and multiprogramming were simply too much
for such a tiny system to handle. As the microcomputers evolved into modern personal computers, they acquired the necessary hardware and then the necessary software to handle more advanced features. It is likely that this development will continue for years to come. Other fields may also have this wheel of reincarnation, but
in the computer industry it seems to spin faster.

### **Disks**

Early mainframes were largely magnetic-tape based. They would read in a program from tape, compile it, run it, and write the results back to another tape. There
were no disks and no concept of a file system. That began to change when IBM
introduced the first hard disk—the RAMAC (RAndoM ACcess) in 1956. It occupied about 4 square meters of floor space and could store 5 million 7-bit characters, enough for one medium-resolution digital photo. But with an annual rental fee
of $35,000, assembling enough of them to store the equivalent of a roll of film got
pricey quite fast. But eventually prices came down and primitive file systems were
developed.

Typical of these new dev elopments was the CDC 6600, introduced in 1964 and
for years by far the fastest computer in the world. Users could create so-called
‘‘permanent files’’ by giving them names and hoping that no other user had also
decided that, say, ‘‘data’’ was a suitable name for a file. This was a single-level directory. Eventually, mainframes developed complex hierarchical file systems, perhaps culminating in the MULTICS file system.

As minicomputers came into use, they eventually also had hard disks. The
standard disk on the PDP-11 when it was introduced in 1970 was the RK05 disk,
with a capacity of 2.5 MB, about half of the IBM RAMAC, but it was only about 

<center>INTRODUCTION</center>

40 cm in diameter and 5 cm high. But it, too, had a single-level directory initially.
When microcomputers came out, CP/M was initially the dominant operating system, and it, too, supported just one directory on the (floppy) disk.

### **Virtual Memory**

Virtual memory (discussed in Chap. 3) gives the ability to run programs larger
than the machine’s physical memory by rapidly moving pieces back and forth between RAM and disk. It underwent a similar development, first appearing on
mainframes, then moving to the minis and the micros. Virtual memory also allowed having a program dynamically link in a library at run time instead of having it
compiled in. MULTICS was the first system to allow this. Eventually, the idea
propagated down the line and is now widely used on most UNIX and Windows
systems.

In all these developments, we see ideas invented in one context and later
thrown out when the context changes (assembly-language programming, monoprogramming, single-level directories, etc.) only to reappear in a different context
often a decade later. For this reason in this book we will sometimes look at ideas
and algorithms that may seem dated on today’s gigabyte PCs, but which may soon
come back on embedded computers and smart cards.

# **1.6 SYSTEM CALLS**

We hav e seen that operating systems have two main functions: providing
abstractions to user programs and managing the computer’s resources. For the most
part, the interaction between user programs and the operating system deals with the
former; for example, creating, writing, reading, and deleting files. The resource-management part is largely transparent to the users and done automatically.
Thus, the interface between user programs and the operating system is primarily
about dealing with the abstractions. To really understand what operating systems
do, we must examine this interface closely. The system calls available in the interface vary from one operating system to another (although the underlying concepts
tend to be similar).

We are thus forced to make a choice between (1) vague generalities (‘‘operating systems have system calls for reading files’’) and (2) some specific system
(‘‘UNIX has a read system call with three parameters: one to specify the file, one
to tell where the data are to be put, and one to tell how many bytes to read’’).

We have chosen the latter approach. It’s more work that way, but it gives more
insight into what operating systems really do. Although this discussion specifically
refers to POSIX (International Standard 9945-1), hence also to UNIX, System V,
BSD, Linux, MINIX 3, and so on, most other modern operating systems have system calls that perform the same functions, even if the details differ. Since the actual 

<center>SYSTEM CALLS</center>

mechanics of issuing a system call are highly machine dependent and often must
be expressed in assembly code, a procedure library is provided to make it possible
to make system calls from C programs and often from other languages as well.

It is useful to keep the following in mind. Any single-CPU computer can execute only one instruction at a time. If a process is running a user program in user
mode and needs a system service, such as reading data from a file, it has to execute
a trap instruction to transfer control to the operating system. The operating system
then figures out what the calling process wants by inspecting the parameters. Then
it carries out the system call and returns control to the instruction following the
system call. In a sense, making a system call is like making a special kind of procedure call, only system calls enter the kernel and procedure calls do not.

To make the system-call mechanism clearer, let us take a quick look at the read
system call. As mentioned above, it has three parameters: the first one specifying
the file, the second one pointing to the buffer, and the third one giving the number
of bytes to read. Like nearly all system calls, it is invoked from C programs by calling a library procedure with the same name as the system call: *read.* A call from a C program might look like this:

count = read(fd, buffer, nbytes);

The system call (and the library procedure) return the number of bytes actually
read in *count.* This value is normally the same as *nbytes*, but may be smaller, if,
for example, end-of-file is encountered while reading.

If the system call cannot be carried out owing to an invalid parameter or a disk
error, count is set to −1, and the error number is put in a global variable, errno.
Programs should always check the results of a system call to see if an error occurred.

System calls are performed in a series of steps. To make this concept clearer,
let us examine the read call discussed above. In preparation for calling the read library procedure, which actually makes the read system call, the calling program
first pushes the parameters onto the stack, as shown in steps 1–3 in Fig. 1-17.

C and C++ compilers push the parameters onto the stack in reverse order for
historical reasons (having to do with making the first parameter to printf, the format string, appear on top of the stack). The first and third parameters are called by
value, but the second parameter is passed by reference, meaning that the address of
the buffer (indicated by &) is passed, not the contents of the buffer. Then comes the
actual call to the library procedure (step 4). This instruction is the normal procedure-call instruction used to call all procedures.

The library procedure, possibly written in assembly language, typically puts
the system-call number in a place where the operating system expects it, such as a
register (step 5). Then it executes a TRAP instruction to switch from user mode to
kernel mode and start execution at a fixed address within the kernel (step 6). The
TRAP instruction is actually fairly similar to the procedure-call instruction in the

<center>INTRODUCTION</center>

![](117.jpeg)

sense that the instruction following it is taken from a distant location and the return
address is saved on the stack for use later.

Nevertheless, the TRAP instruction also differs from the procedure-call instruction in two fundamental ways. First, as a side effect, it switches into kernel mode.
The procedure call instruction does not change the mode. Second, rather than giving a relative or absolute address where the procedure is located, the TRAP instruction cannot jump to an arbitrary address. Depending on the architecture, either it
jumps to a single fixed location or there is an 8-bit field in the instruction giving
the index into a table in memory containing jump addresses, or equivalent.

The kernel code that starts following the TRAP examines the system-call number and then dispatches to the correct system-call handler, usually via a table of
pointers to system-call handlers indexed on system-call number (step 7). At that
point the system-call handler runs (step 8). Once it has completed its work, control
may be returned to the user-space library procedure at the instruction following the
TRAP instruction (step 9). This procedure then returns to the user program in the
usual way procedure calls return (step 10).

To finish the job, the user program has to clean up the stack, as it does after
any procedure call (step 11). Assuming the stack grows downward, as it often 

<center>SYSTEM CALLS</center>

does, the compiled code increments the stack pointer exactly enough to remove the
parameters pushed before the call to read. The program is now free to do whatever
it wants to do next.

In step 9 above, we said ‘‘may be returned to the user-space library procedure’’
for good reason. The system call may block the caller, preventing it from continuing. For example, if it is trying to read from the keyboard and nothing has been
typed yet, the caller has to be blocked. In this case, the operating system will look
around to see if some other process can be run next. Later, when the desired input
is available, this process will get the attention of the system and run steps 9–11.

In the following sections, we will examine some of the most heavily used
POSIX system calls, or more specifically, the library procedures that make those
system calls. POSIX has about 100 procedure calls. Some of the most important
ones are listed in Fig. 1-18, grouped for convenience in four categories. In the text
we will briefly examine each call to see what it does.

To a large extent, the services offered by these calls determine most of what
the operating system has to do, since the resource management on personal computers is minimal (at least compared to big machines with multiple users). The
services include things like creating and terminating processes, creating, deleting,
reading, and writing files, managing directories, and performing input and output.

As an aside, it is worth pointing out that the mapping of POSIX procedure
calls onto system calls is not one-to-one. The POSIX standard specifies a number
of procedures that a conformant system must supply, but it does not specify whether they are system calls, library calls, or something else. If a procedure can be carried out without invoking a system call (i.e., without trapping to the kernel), it will
usually be done in user space for reasons of performance. However, most of the
POSIX procedures do invoke system calls, usually with one procedure mapping directly onto one system call. In a few cases, especially where several required procedures are only minor variations of one another, one system call handles more
than one library call.

### **1.6.1 System Calls for Proses Management**

The first group of calls in Fig. 1-18 deals with process management. Fork is a
good place to start the discussion. Fork is the only way to create a new process in
POSIX. It creates an exact duplicate of the original process, including all the file
descriptors, registers—everything. After the fork, the original process and the copy
(the parent and child) go their separate ways. All the variables have identical values at the time of the fork, but since the parent’s data are copied to create the child,
subsequent changes in one of them do not affect the other one. (The program text,
which is unchangeable, is shared between parent and child.) The fork call returns a
value, which is zero in the child and equal to the child’s **PID (Process IDentifier)**
in the parent. Using the returned PID, the two processes can see which one is the
parent process and which one is the child process.

<center>INTRODUCTION</center>

<center>Process management</center>

![](118.jpeg)

In most cases, after a fork, the child will need to execute different code from
the parent. Consider the case of the shell. It reads a command from the terminal,
forks off a child process, waits for the child to execute the command, and then
reads the next command when the child terminates. To wait for the child to finish, 

<center>SYSTEM CALLS</center>

the parent executes a waitpid system call, which just waits until the child terminates
(any child if more than one exists). Waitpid can wait for a specific child, or for any
old child by setting the first parameter to −1. When waitpid completes, the address
pointed to by the second parameter, statloc, will be set to the child process’ exit
status (normal or abnormal termination and exit value). Various options are also
provided, specified by the third parameter. For example, returning immediately if
no child has already exited.

Now consider how fork is used by the shell. When a command is typed, the
shell forks off a new process. This child process must execute the user command.
It does this by using the execve system call, which causes its entire core image to
be replaced by the file named in its first parameter. (Actually, the system call itself
is exec, but several library procedures call it with different parameters and slightly
different names. We will treat these as system calls here.) A highly simplified shell
illustrating the use of fork, waitpid, and execve is shown in Fig. 1-19.

![](119.jpeg)

In the most general case, execve has three parameters: the name of the file to
be executed, a pointer to the argument array, and a pointer to the environment
array. These will be described shortly. Various library routines, including execl,
execv, execle, and execve, are provided to allow the parameters to be omitted or
specified in various ways. Throughout this book we will use the name exec to
represent the system call invoked by all of these.

Let us consider the case of a command such as

cp file1 file2

used to copy file1 to file2. After the shell has forked, the child process locates and
executes the file cp and passes to it the names of the source and target files.

<center>INTRODUCTION</center>

The main program of cp (and main program of most other C programs) contains the declaration

main(argc, argv, envp)

where *argc* is a count of the number of items on the command line, including the
program name. For the example above, *argc* is 3.

The second parameter, *argv*, is a pointer to an array. Element i of that array is a
pointer to the ith string on the command line. In our example, *argv*[0] would point
to the string ‘‘cp’’, *argv*[1] would point to the string ‘‘file1’’, and *argv*[2] would
point to the string ‘‘file2’’.

The third parameter of main, envp, is a pointer to the environment, an array of
strings containing assignments of the form name = value used to pass information
such as the terminal type and home directory name to programs. There are library
procedures that programs can call to get the environment variables, which are often
used to customize how a user wants to perform certain tasks (e.g., the default printer to use). In Fig. 1-19, no environment is passed to the child, so the third parameter of execve is a zero.

If exec seems complicated, do not despair; it is (semantically) the most complex of all the POSIX system calls. All the other ones are much simpler. As an example of a simple one, consider exit, which processes should use when they are
finished executing. It has one parameter, the exit status (0 to 255), which is returned to the parent via statloc in the waitpid system call.

Processes in UNIX have their memory divided up into three segments: the **text segment** (i.e., the program code), the **data segment** (i.e., the variables), and the
**stack segment**. The data segment grows upward and the stack grows downward,
as shown in Fig. 1-20. Between them is a gap of unused address space. The stack
grows into the gap automatically, as needed, but expansion of the data segment is
done explicitly by using a system call, br k, which specifies the new address where
the data segment is to end. This call, however, is not defined by the POSIX standard, since programmers are encouraged to use the malloc library procedure for
dynamically allocating storage, and the underlying implementation of malloc was
not thought to be a suitable subject for standardization since few programmers use
it directly and it is doubtful that anyone even notices that br k is not in POSIX.

### **1.6.2 System Calls for File Management**

Many system calls relate to the file system. In this section we will look at calls
that operate on individual files; in the next one we will examine those that involve
directories or the file system as a whole.

To read or write a file, it must first be opened. This call specifies the file name
to be opened, either as an absolute path name or relative to the working directory,
as well as a code of O RDONLY, O WRONLY, or O RDWR, meaning open for
reading, writing, or both. To create a new file, the O CREAT parameter is used.

<center> SYSTEM CALLS</center>

![](120.jpeg)

The file descriptor returned can then be used for reading or writing. Afterward, the file can be closed by close, which makes the file descriptor available for reuse on a subsequent open.

The most heavily used calls are undoubtedly read and wr ite. We saw read earlier. Write has the same parameters.

Although most programs read and write files sequentially, for some applications programs need to be able to access any part of a file at random. Associated
with each file is a pointer that indicates the current position in the file. When reading (writing) sequentially, it normally points to the next byte to be read (written).
The lseek call changes the value of the position pointer, so that subsequent calls to read or write can begin anywhere in the file.


Lseek has three parameters: the first is the file descriptor for the file, the second is a file position, and the third tells whether the file position is relative to the
beginning of the file, the current position, or the end of the file. The value returned
by lseek is the absolute position in the file (in bytes) after changing the pointer.


For each file, UNIX keeps track of the file mode (regular file, special file, directory, and so on), size, time of last modification, and other information. Programs can ask to see this information via the stat system call. The first parameter
specifies the file to be inspected; the second one is a pointer to a structure where the information is to be put. The fstat calls does the samething for an open file.

### **1.6.3 System Calls for Directory Management** 


In this section we will look at some system calls that relate more to directories
or the file system as a whole, rather than just to one specific file as in the previous
section. The first two calls, mkdir and rmdir, create and remove empty directories,
respectively. The next call is link. Its purpose is to allow the same file to appear
under two or more names, often in different directories. A typical use is to allow
several members of the same programming team to share a common file, with each
of them having the file appear in his own directory, possibly under different names.
Sharing a file is not the same as giving every team member a private copy; having 

<center>INTRODUCTION</center>
a shared file means that changes that any member of the team makes are instantly
visible to the other members—there is only one file. When copies are made of a file, subsequent changes made to one copy do not affect the others.

To see how link works, consider the situation of Fig. 1-21(a). Here are two
users, ast and jim, each having his own directory with some files. If ast now executes a program containing the system call

link("/usr/jim/memo", "/usr/ast/note");

*the file memo in jim’s directory is now entered into ast’s directory under the name note. Thereafter, /usr/jim/memo and /usr/ast/note refer to the same file. As an aside, whether user directories are kept in /usr, /user, /home, or somewhere else is simply a decision made by the local system administrator.*

![](121.jpeg)

Understanding how link works will probably make it clearer what it does.
Every file in UNIX has a unique number, its i-number, that identifies it. This i-number is an index into a table of **i-nodes**, one per file, telling who owns the file, where its disk blocks are, and so on. A directory is simply a file containing a set of
(i-number, ASCII name) pairs. In the first versions of UNIX, each directory entry
was 16 bytes—2 bytes for the i-number and 14 bytes for the name. Now a more
complicated structure is needed to support long file names, but conceptually a directory is still a set of (i-number, ASCII name) pairs. In Fig. 1-21, mail has i-number 16, and so on. What link does is simply create a brand new directory entry with
a (possibly new) name, using the i-number of an existing file. In Fig. 1-21(b), two entries have the same i-number (70) and thus refer to the same file. If either one is later removed, using the unlink system call, the other one remains. If both are removed, UNIX sees that no entries to the file exist (a field in the i-node keeps track of the number of directory entries pointing to the file), so the file is removed from
the disk.

As we have mentioned earlier, the mount system call allows two file systems to be merged into one. A common situation is to have the root file system, containing
the binary (executable) versions of the common commands and other heavily used files, on a hard disk (sub)partition and user files on another (sub)partition. Further,
the user can then insert a USB disk with files to be read.

<center>SYSTEM CALLS</center>

By executing the mount system call, the USB file system can be attached to the
root file system, as shown in Fig. 1-22. A typical statement in C to mount is

mount("/dev/sdb0", "/mnt", 0);

where the first parameter is the name of a block special file for USB drive 0, the
second parameter is the place in the tree where it is to be mounted, and the third
parameter tells whether the file system is to be mounted read-write or read-only.

![](122.jpeg)

After the mount call, a file on drive 0 can be accessed by just using its path from the root directory or the working directory, without regard to which drive it is on. In fact, second, third, and fourth drives can also be mounted anywhere in the tree. The mount call makes it possible to integrate removable media into a single
integrated file hierarchy, without having to worry about which device a file is on.
Although this example involves CD-ROMs, portions of hard disks (often called
**partitions** or **minor devices**) can also be mounted this way, as well as external
hard disks and USB sticks. When a file system is no longer needed, it can be
unmounted with the umount system call.

### **1.6.4 Miscellaneous System Calls**

A variety of other system calls exist as well. We will look at just four of them
here. The chdir call changes the current working directory. After the call

chdir("/usr/ast/test");

an open on the file xyz will open /usr/ast/test/xyz. The concept of a working directory eliminates the need for typing (long) absolute path names all the time.

y eliminates the need for typing (long) absolute path names all the time.
In UNIX every file has a mode used for protection. The mode includes the
read-write-execute bits for the owner, group, and others. The chmod system call
makes it possible to change the mode of a file. For example, to make a file readonly by everyone except the owner, one could execute

chmod("file", 0644);

The kill system call is the way users and user processes send signals. If a process is prepared to catch a particular signal, then when it arrives, a signal handler is run. 

<center>INTRODUCTION</center>

If the process is not prepared to handle a signal, then its arrival kills the process (hence the name of the call).

POSIX defines a number of procedures for dealing with time. For example,
time just returns the current time in seconds, with 0 corresponding to Jan. 1, 1970
at midnight (just as the day was starting, not ending). On computers using 32-bit
words, the maximum value time can return is 232 − 1 seconds (assuming an unsigned integer is used). This value corresponds to a little over 136 years. Thus in the
year 2106, 32-bit UNIX systems will go berserk, not unlike the famous Y2K problem that would have wreaked havoc with the world’s computers in 2000, were it
not for the massive effort the IT industry put into fixing the problem. If you currently have a 32-bit UNIX system, you are advised to trade it in for a 64-bit one
sometime before the year 2106.

### **1.6.5 The Windows Win32 API**

So far we have focused primarily on UNIX. Now it is time to look briefly at
Windows. Windows and UNIX differ in a fundamental way in their respective programming models. A UNIX program consists of code that does something or
other, making system calls to have certain services performed. In contrast, a Windows program is normally event driven. The main program waits for some event to
happen, then calls a procedure to handle it. Typical events are keys being struck,
the mouse being moved, a mouse button being pushed, or a USB drive inserted.
Handlers are then called to process the event, update the screen and update the internal program state. All in all, this leads to a somewhat different style of programming than with UNIX, but since the focus of this book is on operating system
function and structure, these different programming models will not concern us
much more.

Of course, Windows also has system calls. With UNIX, there is almost a oneto-one relationship between the system calls (e.g., read) and the library procedures
(e.g., read) used to invoke the system calls. In other words, for each system call,
there is roughly one library procedure that is called to invoke it, as indicated in
Fig. 1-17. Furthermore, POSIX has only about 100 procedure calls.

With Windows, the situation is radically different. To start with, the library
calls and the actual system calls are highly decoupled. Microsoft has defined a set
of procedures called the **Win32 API (Application Programming Interface)** that
programmers are expected to use to get operating system services. This interface is
(partially) supported on all versions of Windows since Windows 95. By decoupling the API interface from the actual system calls, Microsoft retains the ability to
change the actual system calls in time (even from release to release) without invalidating existing programs. What actually constitutes Win32 is also slightly ambiguous because recent versions of Windows have many new calls that were not previously available. In this section, Win32 means the interface supported by all versions of Windows. Win32 provides compatibility among versions of Windows.

<center>SYSTEM CALLS</center>

The number of Win32 API calls is extremely large, numbering in the thousands. Furthermore, while many of them do invoke system calls, a substantial number are carried out entirely in user space. As a consequence, with Windows it is
impossible to see what is a system call (i.e., performed by the kernel) and what is
simply a user-space library call. In fact, what is a system call in one version of
Windows may be done in user space in a different version, and vice versa. When
we discuss the Windows system calls in this book, we will use the Win32 procedures (where appropriate) since Microsoft guarantees that these will be stable over
time. But it is worth remembering that not all of them are true system calls (i.e.,
traps to the kernel).

The Win32 API has a huge number of calls for managing windows, geometric
figures, text, fonts, scrollbars, dialog boxes, menus, and other features of the GUI.
To the extent that the graphics subsystem runs in the kernel (true on some versions
of Windows but not on all), these are system calls; otherwise they are just library
calls. Should we discuss these calls in this book or not? Since they are not really
related to the function of an operating system, we have decided not to, even though
they may be carried out by the kernel. Readers interested in the Win32 API should
consult one of the many books on the subject (e.g., Hart, 1997; Rector and Newcomer, 1997; and Simon, 1997).

Even introducing all the Win32 API calls here is out of the question, so we will
restrict ourselves to those calls that roughly correspond to the functionality of the
UNIX calls listed in Fig. 1-18. These are listed in Fig. 1-23.

Let us now briefly go through the list of Fig. 1-23. CreateProcess creates a
new process. It does the combined work of fork and execve in UNIX. It has many
parameters specifying the properties of the newly created process. Windows does
not have a process hierarchy as UNIX does so there is no concept of a parent process and a child process. After a process is created, the creator and createe are
equals. WaitForSingleObject is used to wait for an event. Many possible events can
be waited for. If the parameter specifies a process, then the caller waits for the
specified process to exit, which is done using ExitProcess.

pecified process to exit, which is done using ExitProcess.
The next six calls operate on files and are functionally similar to their UNIX
counterparts although they differ in the parameters and details. Still, files can be
opened, closed, read, and written pretty much as in UNIX. The SetFilePointer and
GetFileAttr ibutesEx calls set the file position and get some of the file attributes.
Windows has directories and they are created with CreateDirector y and
RemoveDirector y API calls, respectively. There is also a notion of a current directory, set by SetCurrentDirector y. The current time of day is acquired using GetLocalTime.

The Win32 interface does not have links to files, mounted file systems, security, or signals, so the calls corresponding to the UNIX ones do not exist. Of course,
Win32 has a huge number of other calls that UNIX does not have, especially for
managing the GUI. Windows Vista has an elaborate security system and also supports file links. Windows 7 and 8 add yet more features and system calls.

<center>INTRODUCTION</center>

![](123.jpeg)

One last note about Win32 is perhaps worth making. Win32 is not a terribly
uniform or consistent interface. The main culprit here was the need to be backward compatible with the previous 16-bit interface used in Windows 3.x.

# **1.7 OPERATING STYSTEM STRUCTURE**

Now that we have seen what operating systems look like on the outside (i.e.,
the programmer’s interface), it is time to take a look inside. In the following sections, we will examine six different structures that have been tried, in order to get
some idea of the spectrum of possibilities. These are by no means exhaustive, but
they giv e an idea of some designs that have been tried in practice. The six designs
we will discuss here are monolithic systems, layered systems, microkernels, client-server systems, virtual machines, and exokernels.

<center>OPERATING SYSTEM STRUCTURE</center>

### **1.7.1 Monolithic Systems**

By far the most common organization, in the monolithic approach the entire
operating system runs as a single program in kernel mode. The operating system is
written as a collection of procedures, linked together into a single large executable
binary program. When this technique is used, each procedure in the system is free
to call any other one, if the latter provides some useful computation that the former
needs. Being able to call any procedure you want is very efficient, but having thousands of procedures that can call each other without restriction may also lead to a
system that is unwieldy and difficult to understand. Also, a crash in any of these
procedures will take down the entire operating system.

To construct the actual object program of the operating system when this approach is used, one first compiles all the individual procedures (or the files containing the procedures) and then binds them all together into a single executable
file using the system linker. In terms of information hiding, there is essentially
none—every procedure is visible to every other procedure (as opposed to a structure containing modules or packages, in which much of the information is hidden
aw ay inside modules, and only the officially designated entry points can be called
from outside the module).

Even in monolithic systems, however, it is possible to have some structure. The
services (system calls) provided by the operating system are requested by putting
the parameters in a well-defined place (e.g., on the stack) and then executing a trap
instruction. This instruction switches the machine from user mode to kernel mode
and transfers control to the operating system, shown as step 6 in Fig. 1-17. The
operating system then fetches the parameters and determines which system call is
to be carried out. After that, it indexes into a table that contains in slot k a pointer
to the procedure that carries out system call k (step 7 in Fig. 1-17).
This organization suggests a basic structure for the operating system:


1. A main program that invokes the requested service procedure.
2. A set of service procedures that carry out the system calls.
3. A set of utility procedures that help the service procedures.

In this model, for each system call there is one service procedure that takes care of
it and executes it. The utility procedures do things that are needed by several service procedures, such as fetching data from user programs. This division of the
procedures into three layers is shown in Fig. 1-24.

In addition to the core operating system that is loaded when the computer is
booted, many operating systems support loadable extensions, such as I/O device
drivers and file systems. These components are loaded on demand. In UNIX they
are called **shared libraries**. In Windows they are called **DLLs (Dynamic-Link Libraries)**. They hav e file extension .dll and the C:\Windows\system32 directory
on Windows systems has well over 1000 of them.

<center>INTRODUCTION </center>

![](124.jpeg)


### **1.7.2 Layered System**

A generalization of the approach of Fig. 1-24 is to organize the operating system as a hierarchy of layers, each one constructed upon the one below it. The first
system constructed in this way was the THE system built at the Technische Hogeschool Eindhoven in the Netherlands by E. W. Dijkstra (1968) and his students.
The THE system was a simple batch system for a Dutch computer, the Electrologica X8, which had 32K of 27-bit words (bits were expensive back then).

The system had six layers, as shown in Fig. 1-25. Layer 0 dealt with allocation of the processor, switching between processes when interrupts occurred or timers
expired. Above layer 0, the system consisted of sequential processes, each of which could be programmed without having to worry about the fact that multiple processes were running on a single processor. In other words, layer 0 provided the
basic multiprogramming of the CPU.

![](125.jpeg)

Layer 1 did the memory management. It allocated space for processes in main
memory and on a 512K word drum used for holding parts of processes (pages) for
which there was no room in main memory. Above layer 1, processes did not have
to worry about whether they were in memory or on the drum; the layer 1 software 

<center>OPERATING SYSTEM STRUCTURE</center>

took care of making sure pages were brought into memory at the moment they
were needed and removed when they were not needed.

Layer 2 handled communication between each process and the operator console (that is, the user). On top of this layer each process effectively had its own operator console. Layer 3 took care of managing the I/O devices and buffering the
information streams to and from them. Above layer 3 each process could deal with
abstract I/O devices with nice properties, instead of real devices with many peculiarities. Layer 4 was where the user programs were found. They did not have to
worry about process, memory, console, or I/O management. The system operator process was located in layer 5.

A further generalization of the layering concept was present in the MULTICS
system. Instead of layers, MULTICS was described as having a series of concentric
rings, with the inner ones being more privileged than the outer ones (which is effectively the same thing). When a procedure in an outer ring wanted to call a procedure in an inner ring, it had to make the equivalent of a system call, that is, a
TRAP instruction whose parameters were carefully checked for validity before the
call was allowed to proceed. Although the entire operating system was part of the address space of each user process in MULTICS, the hardware made it possible to
designate individual procedures (memory segments, actually) as protected against
reading, writing, or executing.

Whereas the THE layering scheme was really only a design aid, because all the parts of the system were ultimately linked together into a single executable program, in MULTICS, the ring mechanism was very much present at run time and
enforced by the hardware. The advantage of the ring mechanism is that it can easily be extended to structure user subsystems. For example, a professor could write a
program to test and grade student programs and run this program in ring n, with
the student programs running in ring n + 1 so that they could not change their
grades.

### **1.7.3 Microkernels**

With the layered approach, the designers have a choice where to draw the kernel-user boundary. Traditionally, all the layers went in the kernel, but that is not
necessary. In fact, a strong case can be made for putting as little as possible in kernel mode because bugs in the kernel can bring down the system instantly. In contrast, user processes can be set up to have less power so that a bug there may not be
fatal.

Various researchers have repeatedly studied the number of bugs per 1000 lines of code (e.g., Basilli and Perricone, 1984; and Ostrand and Weyuker, 2002). Bug density depends on module size, module age, and more, but a ballpark figure for serious industrial systems is between two and ten bugs per thousand lines of code. This means that a monolithic operating system of fiv e million lines of code is likely to contain between 10,000 and 50,000 kernel bugs. Not all of these are fatal, of 

<center>INTRODUCTION</center>
course, since some bugs may be things like issuing an incorrect error message in a
situation that rarely occurs. Nevertheless, operating systems are sufficiently buggy that computer manufacturers put reset buttons on them (often on the front panel),
something the manufacturers of TV sets, stereos, and cars do not do, despite the large amount of software in these devices.

The basic idea behind the microkernel design is to achieve high reliability by splitting the operating system up into small, well-defined modules, only one of
which—the microkernel—runs in kernel mode and the rest run as relatively powerless ordinary user processes. In particular, by running each device driver and file system as a separate user process, a bug in one of these can crash that component,
but cannot crash the entire system. Thus a bug in the audio driver will cause the
sound to be garbled or stop, but will not crash the computer. In contrast, in a
monolithic system with all the drivers in the kernel, a buggy audio driver can easily
reference an invalid memory address and bring the system to a grinding halt instantly.

Many microkernels have been implemented and deployed for decades (Haertig
et al., 1997; Heiser et al., 2006; Herder et al., 2006; Hildebrand, 1992; Kirsch et
al., 2005; Liedtke, 1993, 1995, 1996; Pike et al., 1992; and Zuberi et al., 1999).
With the exception of OS X, which is based on the Mach microkernel (Accetta et
al., 1986), common desktop operating systems do not use microkernels. However,
they are dominant in real-time, industrial, avionics, and military applications that
are mission critical and have very high reliability requirements. A few of the better-known microkernels include Integrity, K42, L4, PikeOS, QNX, Symbian, and
MINIX 3. We now giv e a brief overview of MINIX 3, which has taken the idea of
modularity to the limit, breaking most of the operating system up into a number of
independent user-mode processes. MINIX 3 is a POSIX-conformant, open source
system freely available at www.minix3.org (Giuffrida et al., 2012; Giuffrida et al.,
2013; Herder et al., 2006; Herder et al., 2009; and Hruby et al., 2013).

The MINIX 3 microkernel is only about 12,000 lines of C and some 1400 lines
of assembler for very low-level functions such as catching interrupts and switching
processes. The C code manages and schedules processes, handles interprocess
communication (by passing messages between processes), and offers a set of about
40 kernel calls to allow the rest of the operating system to do its work. These calls
perform functions like hooking handlers to interrupts, moving data between address spaces, and installing memory maps for new processes. The process structure
of MINIX 3 is shown in Fig. 1-26, with the kernel call handlers labeled Sys. The
device driver for the clock is also in the kernel because the scheduler interacts
closely with it. The other device drivers run as separate user processes.

Outside the kernel, the system is structured as three layers of processes all running in user mode. The lowest layer contains the device drivers. Since they run in
user mode, they do not have physical access to the I/O port space and cannot issue
I/O commands directly. Instead, to program an I/O device, the driver builds a structure telling which values to write to which I/O ports and makes a kernel call telling 

<center>OPERATING SYSTEM STRUCTURE</center>

![](126.jpeg)

the kernel to do the write. This approach means that the kernel can check to see
that the driver is writing (or reading) from I/O it is authorized to use. Consequently
(and unlike a monolithic design), a buggy audio driver cannot accidentally write on
the disk.

Above the drivers is another user-mode layer containing the servers, which do
most of the work of the operating system. One or more file servers manage the file
system(s), the process manager creates, destroys, and manages processes, and so
on. User programs obtain operating system services by sending short messages to
the servers asking for the POSIX system calls. For example, a process needing to
do a read sends a message to one of the file servers telling it what to read.

One interesting server is the **reincarnation server**, whose job is to check if the
other servers and drivers are functioning correctly. In the event that a faulty one is
detected, it is automatically replaced without any user intervention. In this way,
the system is self healing and can achieve high reliability.

The system has many restrictions limiting the power of each process. As mentioned, drivers can touch only authorized I/O ports, but access to kernel calls is also
controlled on a per-process basis, as is the ability to send messages to other processes. Processes can also grant limited permission for other processes to have the
kernel access their address spaces. As an example, a file system can grant permission for the disk driver to let the kernel put a newly read-in disk block at a specific
address within the file system’s address space. The sum total of all these restrictions is that each driver and server has exactly the power to do its work and nothing
more, thus greatly limiting the damage a buggy component can do.

An idea somewhat related to having a minimal kernel is to put the mechanism
for doing something in the kernel but not the policy. To make this point better,
consider the scheduling of processes. A relatively simple scheduling algorithm is
to assign a numerical priority to every process and then have the kernel run the 

<center>INTRODUCTION</center>

highest-priority process that is runnable. The mechanism—in the kernel—is to
look for the highest-priority process and run it. The policy—assigning priorities to
processes—can be done by user-mode processes. In this way, policy and mechanism can be decoupled and the kernel can be made smaller.

### **1.7.4 Client-Server Model**

A slight variation of the microkernel idea is to distinguish two classes of processes, the servers, each of which provides some service, and the clients, which use
these services. This model is known as the client-server model. Often the lowest
layer is a microkernel, but that is not required. The essence is the presence of client processes and server processes.

Communication between clients and servers is often by message passing. To
obtain a service, a client process constructs a message saying what it wants and
sends it to the appropriate service. The service then does the work and sends back
the answer. If the client and server happen to run on the same machine, certain
optimizations are possible, but conceptually, we are still talking about message
passing here.

An obvious generalization of this idea is to have the clients and servers run on
different computers, connected by a local or wide-area network, as depicted in
Fig. 1-27. Since clients communicate with servers by sending messages, the clients need not know whether the messages are handled locally on their own machines, or whether they are sent across a network to servers on a remote machine.
As far as the client is concerned, the same thing happens in both cases: requests are
sent and replies come back. Thus the client-server model is an abstraction that can
be used for a single machine or for a network of machines.

![](127.jpeg)

Increasingly many systems involve users at their home PCs as clients and large machines elsewhere running as servers. In fact, much of the Web operates this way. A PC sends a request for a Web page to the server and the Web page comes back. This is a typical use of the client-server model in a network.

<center> OPERATING SYSTEM STRUCTURE</center>

### **1.7.5 Virtual Machines**

The initial releases of OS/360 were strictly batch systems. Nevertheless, many 360 users wanted to be able to work interactively at a terminal, so various groups, both inside and outside IBM, decided to write timesharing systems for it. The official IBM timesharing system, TSS/360, was delivered late, and when it finally arrived it was so big and slow that few sites converted to it. It was eventually abandoned after its development had consumed some $50 million (Graham, 1970). But
a group at IBM’s Scientific Center in Cambridge, Massachusetts, produced a radically different system that IBM eventually accepted as a product. A linear descendant of it, called **z/VM**, is now widely used on IBM’s current mainframes, the zSeries, which are heavily used in large corporate data centers, for example, as e-commerce servers that handle hundreds or thousands of transactions per second
and use databases whose sizes run to millions of gigabytes.

### **VM/370**

This system, originally called CP/CMS and later renamed VM/370 (Seawright
and MacKinnon, 1979), was based on an astute observation: a timesharing system
provides (1) multiprogramming and (2) an extended machine with a more convenient interface than the bare hardware. The essence of VM/370 is to completely
separate these two functions.

The heart of the system, known as the **virtual machine monitor**, runs on the
bare hardware and does the multiprogramming, providing not one, but several virtual machines to the next layer up, as shown in Fig. 1-28. However, unlike all
other operating systems, these virtual machines are not extended machines, with
files and other nice features. Instead, they are exact copies of the bare hardware, including kernel/user mode, I/O, interrupts, and everything else the real machine has.

![](128.jpeg)

Because each virtual machine is identical to the true hardware, each one can run any operating system that will run directly on the bare hardware. Different virtual machines can, and frequently do, run different operating systems. On the original IBM VM/370 system, some ran OS/360 or one of the other large batch or 

<center>INTRODUCTION</center>

transaction-processing operating systems, while others ran a single-user, interactive
system called CMS (Conversational Monitor System) for interactive timesharing
users. The latter was popular with programmers.

When a CMS program executed a system call, the call was trapped to the operating system in its own virtual machine, not to VM/370, just as it would be were it
running on a real machine instead of a virtual one. CMS then issued the normal hardware I/O instructions for reading its virtual disk or whatever was needed to carry out the call. These I/O instructions were trapped by VM/370, which then performed them as part of its simulation of the real hardware. By completely separating the functions of multiprogramming and providing an extended machine, each of the pieces could be much simpler, more flexible, and much easier to maintain.

In its modern incarnation, z/VM is usually used to run multiple complete operating systems rather than stripped-down single-user systems like CMS. For example, the zSeries is capable of running one or more Linux virtual machines along
with traditional IBM operating systems.

### **Virtual Machines Rediscovered**

While IBM has had a virtual-machine product available for four decades, and a
few other companies, including Oracle and Hewlett-Packard, have recently added
virtual-machine support to their high-end enterprise servers, the idea of virtualization has largely been ignored in the PC world until recently. But in the past
few years, a combination of new needs, new software, and new technologies have
combined to make it a hot topic.

First the needs. Many companies have traditionally run their mail servers, Web
servers, FTP servers, and other servers on separate computers, sometimes with different operating systems. They see virtualization as a way to run them all on the
same machine without having a crash of one server bring down the rest.

Virtualization is also popular in the Web hosting world. Without virtualization,
Web hosting customers are forced to choose between **shared hosting** (which just
gives them a login account on a Web server, but no control over the server software) and dedicated hosting (which gives them their own machine, which is very
flexible but not cost effective for small to medium Websites). When a Web hosting
company offers virtual machines for rent, a single physical machine can run many
virtual machines, each of which appears to be a complete machine. Customers who
rent a virtual machine can run whatever operating system and software they want
to, but at a fraction of the cost of a dedicated server (because the same physical
machine supports many virtual machines at the same time).

Another use of virtualization is for end users who want to be able to run two or
more operating systems at the same time, say Windows and Linux, because some
of their favorite application packages run on one and some run on the other. This
situation is illustrated in Fig. 1-29(a), where the term ‘‘virtual machine monitor’’
has been renamed **type 1 hypervisor**, which is commonly used nowadays because ‘‘virtual machine monitor’’ requires more keystrokes than people are prepared to
put up with now. Note that many authors use the terms interchangeably though.

<center>OPERATING SYSTEM STRUCTURE</center>

![](129.jpeg)

While no one disputes the attractiveness of virtual machines today, the problem
then was implementation. In order to run virtual machine software on a computer,
its CPU must be virtualizable (Popek and Goldberg, 1974). In a nutshell, here is
the problem. When an operating system running on a virtual machine (in user
mode) executes a privileged instruction, such as modifying the PSW or doing I/O,
it is essential that the hardware trap to the virtual-machine monitor so the instruction can be emulated in software. On some CPUs—notably the Pentium, its predecessors, and its clones—attempts to execute privileged instructions in user mode
are just ignored. This property made it impossible to have virtual machines on this
hardware, which explains the lack of interest in the x86 world. Of course, there
were interpreters for the Pentium, such as Bochs, that ran on the Pentium, but with
a performance loss of one to two orders of magnitude, they were not useful for serious work.

This situation changed as a result of several academic research projects in the
1990s and early years of this millennium, notably Disco at Stanford (Bugnion et
al., 1997) and Xen at Cambridge University (Barham et al., 2003). These research
papers led to several commercial products (e.g., VMware Workstation and Xen)
and a revival of interest in virtual machines. Besides VMware and Xen, popular
hypervisors today include KVM (for the Linux kernel), VirtualBox (by Oracle),
and Hyper-V (by Microsoft).

Some of these early research projects improved the performance over interpreters like Bochs by translating blocks of code on the fly, storing them in an internal cache, and then reusing them if they were executed again. This improved the
performance considerably, and led to what we will call **machine simulators**, as
shown in Fig. 1-29(b). However, although this technique, known as **binary translation**, helped improve matters, the resulting systems, while good enough to publish papers about in academic conferences, were still not fast enough to use in
commercial environments where performance matters a lot.

<center>INTRODUCTION</center>

The next step in improving performance was to add a kernel module to do
some of the heavy lifting, as shown in Fig. 1-29(c). In practice now, all commercially available hypervisors, such as VMware Workstation, use this hybrid strategy
(and have many other improvements as well). They are called type 2 hypervisors
by everyone, so we will (somewhat grudgingly) go along and use this name in the
rest of this book, even though we would prefer to called them type 1.7 hypervisors
to reflect the fact that they are not entirely user-mode programs. In Chap. 7, we
will describe in detail how VMware Workstation works and what the various
pieces do.

In practice, the real distinction between a type 1 hypervisor and a type 2 hypervisor is that a type 2 makes uses of a **host operating system** and its file system to
create processes, store files, and so on. A type 1 hypervisor has no underlying support and must perform all these functions itself.

After a type 2 hypervisor is started, it reads the installation CD-ROM (or CDROM image file) for the chosen **guest operating system** and installs the guest OS
on a virtual disk, which is just a big file in the host operating system’s file system.
Type 1 hypervisors cannot do this because there is no host operating system to
store files on. They must manage their own storage on a raw disk partition.

When the guest operating system is booted, it does the same thing it does on
the actual hardware, typically starting up some background processes and then a
GUI. To the user, the guest operating system behaves the same way it does when
running on the bare metal even though that is not the case here.

A different approach to handling control instructions is to modify the operating
system to remove them. This approach is not true virtualization, but paravirtualization. We will discuss virtualization in more detail in Chap. 7.

### **The Jav a Virtual Machine**

Another area where virtual machines are used, but in a somewhat different
way, is for running Java programs. When Sun Microsystems invented the Java programming language, it also invented a virtual machine (i.e., a computer architecture) called the **JVM (Java Virtual Machine)**. The Java compiler produces code for JVM, which then typically is executed by a software JVM interpreter. The advantage of this approach is that the JVM code can be shipped over the Internet to
any computer that has a JVM interpreter and run there. If the compiler had produced SPARC or x86 binary programs, for example, they could not have been shipped and run anywhere as easily. (Of course, Sun could have produced a compiler that produced SPARC binaries and then distributed a SPARC interpreter, but
JVM is a much simpler architecture to interpret.) Another advantage of using JVM
is that if the interpreter is implemented properly, which is not completely trivial,
incoming JVM programs can be checked for safety and then executed in a protected environment so they cannot steal data or do any damage.

<center>OPERATING SYSTEM STRUCTURE</center>

### **1.7.6 Exokernels**

Rather than cloning the actual machine, as is done with virtual machines, another strategy is partitioning it, in other words, giving each user a subset of the resources. Thus one virtual machine might get disk blocks 0 to 1023, the next one
might get blocks 1024 to 2047, and so on.

At the bottom layer, running in kernel mode, is a program called the exokernel
(Engler et al., 1995). Its job is to allocate resources to virtual machines and then
check attempts to use them to make sure no machine is trying to use somebody
else’s resources. Each user-level virtual machine can run its own operating system,
as on VM/370 and the Pentium virtual 8086s, except that each one is restricted to
using only the resources it has asked for and been allocated.

The advantage of the exokernel scheme is that it saves a layer of mapping. In
the other designs, each virtual machine thinks it has its own disk, with blocks running from 0 to some maximum, so the virtual machine monitor must maintain
tables to remap disk addresses (and all other resources). With the exokernel, this
remapping is not needed. The exokernel need only keep track of which virtual machine has been assigned which resource. This method still has the advantage of
separating the multiprogramming (in the exokernel) from the user operating system
code (in user space), but with less overhead, since all the exokernel has to do is
keep the virtual machines out of each other’s hair.

# **1.8 THE WORLD ACCORDING TO C**

Operating systems are normally large C (or sometimes C++) programs consisting of many pieces written by many programmers. The environment used for
developing operating systems is very different from what individuals (such as students) are used to when writing small Java programs. This section is an attempt to
give a very brief introduction to the world of writing an operating system for smalltime Java or Python programmers.

### **1.8.1 The C Language**

This is not a guide to C, but a short summary of some of the key differences
between C and languages like **Python** and especially Java. Java is based on C, so
there are many similarities between the two. Python is somewhat different, but still
fairly similar. For convenience, we focus on Java. Java, Python, and C are all
imperative languages with data types, variables, and control statements, for example. The primitive data types in C are integers (including short and long ones),
characters, and floating-point numbers. Composite data types can be constructed
using arrays, structures, and unions. The control statements in C are similar to
those in Java, including if, switch, for, and while statements. Functions and parameters are roughly the same in both languages.

<center>INTRODUCTION</center>

One feature C has that Java and Python do not is explicit pointers. A **pointer**
is a variable that points to (i.e., contains the address of) a variable or data structure.
Consider the statements

char c1, c2, *p;
c1 = ’c’;
p = &c1;
c2 = *p;

which declare c1 and c2 to be character variables and p to be a variable that points
to (i.e., contains the address of) a character. The first assignment stores the ASCII
code for the character ‘‘c’’ in the variable c1. The second one assigns the address
of c1 to the pointer variable p. The third one assigns the contents of the variable
pointed to by p to the variable c2, so after these statements are executed, c2 also
contains the ASCII code for ‘‘c’’. In theory, pointers are typed, so you are not supposed to assign the address of a floating-point number to a character pointer, but in
practice compilers accept such assignments, albeit sometimes with a warning.
Pointers are a very powerful construct, but also a great source of errors when used
carelessly.

Some things that C does not have include built-in strings, threads, packages,
classes, objects, type safety, and garbage collection. The last one is a show stopper
for operating systems. All storage in C is either static or explicitly allocated and
released by the programmer, usually with the library functions malloc and free. It
is the latter property—total programmer control over memory—along with explicit
pointers that makes C attractive for writing operating systems. Operating systems
are basically real-time systems to some extent, even general-purpose ones. When
an interrupt occurs, the operating system may have only a few microseconds to
perform some action or lose critical information. Having the garbage collector kick
in at an arbitrary moment is intolerable.

### **1.8.2 Header Files**

An operating system project generally consists of some number of directories,
each containing many .c files containing the code for some part of the system,
along with some .h header files that contain declarations and definitions used by
one or more code files. Header files can also include simple **macros**, such as

#define BUFFER SIZE 4096

which allows the programmer to name constants, so that when BUFFER SIZE is
used in the code, it is replaced during compilation by the number 4096. Good C
programming practice is to name every constant except 0, 1, and −1, and sometimes even them. Macros can have parameters, such as

#define max(a, b) (a > b ? a : b)

which allows the programmer to write

<center>THE WORLD ACCORDING TO C</center>

i = max(j, k+1)

and get

i = (j > k+1 ? j : k+1)

to store the larger of j and k+1 in i. Headers can also contain conditional compilation, for example

#ifdef X86

intel int ack();

#endif

which compiles into a call to the function intel int ack if the macro X86 is defined
and nothing otherwise. Conditional compilation is heavily used to isolate architecture-dependent code so that certain code is inserted only when the system is compiled on the X86, other code is inserted only when the system is compiled on a
SPARC, and so on. A .c file can bodily include zero or more header files using the
#include directive. There are also many header files that are common to nearly
ev ery .c and are stored in a central directory.

### **1.8.3 Large Programing Projects**

To build the operating system, each .c is compiled into an **object file** by the C
compiler. Object files, which have the suffix .o, contain binary instructions for the
target machine. They will later be directly executed by the CPU. There is nothing
like Java byte code or Python byte code in the C world.

The first pass of the C compiler is called the **C preprocessor**. As it reads each
.c file, every time it hits a #include directive, it goes and gets the header file named
in it and processes it, expanding macros, handling conditional compilation (and
certain other things) and passing the results to the next pass of the compiler as if
they were physically included.

Since operating systems are very large (fiv e million lines of code is not
unusual), having to recompile the entire thing every time one file is changed would
be unbearable. On the other hand, changing a key header file that is included in
thousands of other files does require recompiling those files. Keeping track of
which object files depend on which header files is completely unmanageable without help.

Fortunately, computers are very good at precisely this sort of thing. On UNIX
systems, there is a program called make (with numerous variants such as gmake,
pmake, etc.) that reads the Makefile, which tells it which files are dependent on
which other files. What make does is see which object files are needed to build the
operating system binary and for each one, check to see if any of the files it depends
on (the code and headers) have been modified subsequent to the last time the object file was created. If so, that object file has to be recompiled. When make has
determined which .c files have to recompiled, it then invokes the C compiler to 

<center>INTRODUCTION</center>

recompile them, thus reducing the number of compilations to the bare minimum.
In large projects, creating the Makefile is error prone, so there are tools that do it automatically.

Once all the .o files are ready, they are passed to a program called the linker to
combine all of them into a single executable binary file. Any library functions called are also included at this point, interfunction references are resolved, and machine addresses are relocated as need be. When the linker is finished, the result is
an executable program, traditionally called a.out on UNIX systems. The various
components of this process are illustrated in Fig. 1-30 for a program with three C
files and two header files. Although we have been discussing operating system development here, all of this applies to developing any large program.

![](130.jpeg)

### **1.8.4 The Model of Run Time**

Once the operating system binary has been linked, the computer can be
rebooted and the new operating system started. Once running, it may dynamically
load pieces that were not statically included in the binary such as device drivers 

<center>THE WORLD ACCORDING TO C</center>

and file systems. At run time the operating system may consist of multiple segments, for the text (the program code), the data, and the stack. The text segment is
normally immutable, not changing during execution. The data segment starts out
at a certain size and initialized with certain values, but it can change and grow as
need be. The stack is initially empty but grows and shrinks as functions are called
and returned from. Often the text segment is placed near the bottom of memory,
the data segment just above it, with the ability to grow upward, and the stack segment at a high virtual address, with the ability to grow downward, but different
systems work differently. 

In all cases, the operating system code is directly executed by the hardware,
with no interpreter and no just-in-time compilation, as is normal with Java.

