# 1. Introduction
In this assignment, we will explore buffer overflow vulnerabilities, in the context of a web server called ```zookws```. The zookws web server runs a simple python web application, zoobar, with which users transfer credits between each other. You will find buffer overflows in the zookws web server code, write exploits for the buffer overflows to inject code into the server over the network, and figure out how to bypass non-executable stack protection. 

You may need to familiarize yourself with certain aspects of the C language, x86 assembly language, gdb, etc. as part of this assignment. Detailed familiarity with many different pieces of infrastructure is needed to understand attacks and defenses in realistic situations: security weaknesses often show up in corner cases, and so you need to understand the details to craft exploits and design defenses for those corner cases.  These two factors (new infrastructure and details) can make the labs time consuming. You should start immediately and work on them daily for some limited time, instead of trying to do all exercises in a single shot just before the deadline. Take the time to understand the relevant details.

# 2. Setting Up
To start working on this assignment, you'll need software that lets you run a virtual machine. Linux users can use [KVM](https://www.linux-kvm.org/page/Main_Page)  to an external site., which is built into the Linux kernel. KVM should be available through your distribution, or you can try ```apt-get install qemu-kvm``` KVM requires hardware virtualization support in your CPU, and you must enable this support in your BIOS (which is often, but not always, the default). On other machines, you can use VirtualBox. Once you have virtual machine software installed on your machine, you should download the course VM imageLinks to an external site., and unpack it on your computer. This virtual machine contains an installation of Linux. To start the course VM using VMware, import cs431.vmdk.

Go to Machine > New, select "create virtual machine", choose Linux > Debian 64-bit. Set the memory size to 2048 MB, and use an existing virtual disk (and select the cs431.vmdk file, by searching for it if not shown in the list). Finally, click Create to complete the setup. 

Before starting the VM, make sure that you can ```ssh``` into the machine from outside.

* If you are using VMware you may not require the following setting, and can directly use the NAT adapter to share the IP address with your host VM.
* However, with VirtualBox, one way to get this to work is to select the new VM and go to Machine > Settings > Network, and change Adapter 1 to be attached to "Bridged Adapter".
* If the above does not work with VirtualBox, you may use NAT adapter with port forwarding. To set this up, go to Machine > Settings > Network, and change Adapter 1 to be attached to "NAT". In Advanced section, select Port Forwarding, where you should add two rules. 
<ul><ul><li> The first rule is to forward Host Port 22 to Guest Port 22 - Enter Name as "ssh", Host port as "22" and Guest port at "22". You may leave the other fields blank

<li>The second rule is to forward Host Port 8080 to Guest Port 8080 - Enter Name as "zookws", Host port as "8080" and Guest port at "8080". You may leave the other fields blank
<li> The IPADDRESS in this case would be localhost, i.e., 127.0.0.1
</ul></ul>

## <li> a. With KVM
To start the VM with KVM, run ```./cs431.sh``` Download cs431.shfrom a terminal (Ctrl+A x to force quit). If you get a permission denied error from this script, try adding yourself to the kvm group with sudo gpasswd -a `whoami` kvm, then log out and log back in.

## <li> b. With ARM processors
If you are using a computer with a non-x86 processor (e.g., Mac laptops with the ARM M1 processor), you can run the virtual machine using qemu. To do this, first install [Homebrew](https://brew.sh/) to an external site., then install qemu by running brew install [qemu](https://formulae.brew.sh/formula/qemu) to an external site., and finally edit the cs431.sh Download cs431.shshell script, and remove the ```-enable-kvm flag```. At this point, you should be able to start the course VM by running ```./cs431.sh``` . Download cs431.sh as above.

# 3. Getting Started
You will use the student account in the VM for your work. The password for the student account is cs431. 

The files you will need for this assignment are present in the ```lab``` folder. You will need to upload the code to Git, once you have made the final changes, and can use Git to keep track of any changes you make to the initial source code. Here's the [Git user's manual](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/user-manual.html) to an external site., which you may find useful.

Before you proceed with this lab assignment, make sure you can compile the zookws web server:   
```ps 
student@CS431:~/lab$ make
cc zookd.c -c -o zookd.o -m64 -g -std=c99 -Wall -D_GNU_SOURCE -static -fno-stack-protector
cc http.c -c -o http.o -m64 -g -std=c99 -Wall -D_GNU_SOURCE -static -fno-stack-protector
cc -m64  zookd.o http.o  -lcrypto -o zookd
cc -m64 zookd.o http.o  -lcrypto -o zookd-exstack -z execstack
cc -m64 zookd.o http.o  -lcrypto -o zookd-nxstack
cc zookd.c -c -o zookd-withssp.o -m64 -g -std=c99 -Wall -D_GNU_SOURCE -static
cc http.c -c -o http-withssp.o -m64 -g -std=c99 -Wall -D_GNU_SOURCE -static
cc -m64  zookd-withssp.o http-withssp.o  -lcrypto -o zookd-withssp
cc -m64   -c -o shellcode.o shellcode.S
objcopy -S -O binary -j .text shellcode.o shellcode.bin
cc run-shellcode.c -c -o run-shellcode.o -m64 -g -std=c99 -Wall -D_GNU_SOURCE -static -fno-stack-protector
cc -m64  run-shellcode.o  -lcrypto -o run-shellcode
rm shellcode.o
student@CS431:~/lab$
```
The component of zookws that receives HTTP requests is zookd. It is written in C and serves static files and executes dynamic scripts. For this assignment you don't have to understand the dynamic scripts; they are written in Python and the exploits in this assignment apply only to C code. The HTTP-related code is in http.c. [Here](https://www.garshol.priv.no/download/text/http-tut.html) is a tutorial about the HTTP protocol.

There are two versions of zookd you will be using:

* zookd-exstack
* zookd-nxstack  

zookd-exstack has an executable stack, which makes it easy to inject executable code given a stack buffer overflow vulnerability. zookd-nxstack has a non-executable stack, and requires a more sophisticated attack to exploit stack buffer overflows.

In order to run the web server in a predictable fashion---so that its stack and memory layout is the same every time---you will use the ```clean-env.sh``` script. This is the same way in which we will run the web server during grading, so make sure all of your exploits work on this configuration!

The reference binaries of ```zookd``` are provided in ```bin.tar.gz```, which we will use for grading. Make sure your exploits work on those binaries. The make checkcommand will always use both ```clean-env.sh``` and ```bin.tar.gz``` to check your submission.

Now, make sure you can run the zookws web server and access the zoobar web application from a browser running on your machine, as follows:

```ps 
student@CS431:~/lab$ ./clean-env.sh ./zookd 8080
```

The ```./clean-env.sh``` commands starts zookd on port 8080. To open the zoobar application, open your browser and point it at the ```URL http://IPADDRESS:8080/```, where IPADDRESS is the VM's IP address. To find the VM's IP address, log in on the console, run ip addr show dev eth0, and note the IP address listed beside inet.  

If something doesn't seem to be working, try to figure out what went wrong, or contact us, before proceeding further. You can always ssh into the machine using the IP address obtained from the machine to open more terminals: ```ssh -p 22 student@<IPADDRESS>```

The IPADDRESS may be ```127.0.0.1``` if using NAT adapter with VirtualBox. 

# 4. Problems
Part 1: Finding buffer overflows (2 points)
In the first part of this assignment, you will find buffer overflows in the provided web server. To do this, you will need to understand the basics of buffer overflows. To help you get started with this, you should read [Smashing the Stack in the 21st Century](https://thesquareplanet.com/blog/smashing-the-stack-21st-century/), which goes through the details of how buffer overflows work, and how they can be exploited.

Exercise 1.
----------------
Study the web server's C code (in zookd.c and http.c), and find one example of code that allows an attacker to overwrite the return address of a function. Hint: look for buffers allocated on the stack. Write down a description of the vulnerability in the file answers.txt. For your vulnerability, describe the buffer which may overflow, how you would structure the input to the web server (i.e., the HTTP request) to overflow the buffer and overwrite the return address, and the call stack that will trigger the buffer overflow (i.e., the chain of function calls starting from process_client).

It is worth taking your time on this exercise and familiarizing yourself with the code, because your next job is to exploit the vulnerability you identified. In fact, you may want to go back and forth between this exercise and later exercises, as you work out the details and document them. That is, if you find a buffer overflow that you think can be exploited, you can use later exercises to figure out if it indeed can be exploited. It will be helpful to draw a stack diagram like the figures in [Smashing the Stack in the 21st Century](https://thesquareplanet.com/blog/smashing-the-stack-21st-century/).

Now, you will start developing exploits to take advantage of the buffer overflows you have found above. We have provided template Python code for an exploit in ```/home/student/lab/exploit-template.py```, which issues an HTTP request. The exploit template takes two arguments, the server name and port number, so you might run it as follows to issue a request to zookws running on localhost:
```ps 
student@CS431:~/lab$ ./clean-env.sh ./zookd-exstack 8080 &
[1] ...
student@CS431:~/lab$ ./exploit-template.py localhost 8080
HTTP request:
b'GET / HTTP/1.0

'
...
student@CS431:~/lab$
```
You are free to use this template, or write your own exploit code from scratch. Note, however, that if you choose to write your own exploit, the exploit must run correctly inside the provided virtual machine.

You may find gdb useful in building your exploits (though it is not required for you to do so). As zookd forks off many processes (one for each client), it can be difficult to debug the correct one. The easiest way to do this is to run the web server ahead of time with ```clean-env.sh``` and then attaching ```gdb``` to an already-running process with the -p flag. You can find the PID of a process by using pgrep; for example, to attach to zookd-exstack, start the server and, in another shell, run


```ps
student@CS431:~/lab$ gdb -p $(pgrep zookd-)
...
(gdb) break <your-breakpoint>
Breakpoint 1 at 0x1234567: file zookd.c, line 999.
(gdb) continue
Continuing.
```


Keep in mind that a process being debugged by gdb will not get killed even if you terminate the parent zookd process using ^C. If you are having trouble restarting the web server, check for leftover processes from the previous run, or be sure to exit gdb before restarting zookd. You can also save yourself some typing by using b instead of break, and c instead of continue.

When a process being debugged by gdb forks, by default gdb continues to debug the parent process and does not attach to the child. Since zookd forks a child process to service each request, you may find it helpful to have gdb attach to the child on fork, using the command ```set follow-fork-mode child```. We have added that command to ```/home/student/lab/.gdbinit```, which will take effect if you start gdb in that directory.

As you develop your exploit, you may discover that it causes the server to hang as opposed to crash, depending on what buffer overflow you are trying to take advantage of and what data you are overwriting in the running server. You can dig into the details of why the hang happens, to understand how you are affecting the server's execution, in order to make your exploit avoid the hang and instead crash the server. Or you can choose to exploit a different buffer overflow that avoids the hanging behavior.

For this and subsequent exercises, you may need to encode your attack payload in different ways, depending on which vulnerability you are exploiting. In some cases, you may need to make sure that your attack payload is URL-encoded; that is, use ```+``` instead of space and ```%2b``` instead of ```+```. [Here is a URL encoding reference](http://www.blooberry.com/indexdot/html/topics/urlencoding.htm). and a handy [conversion tool](http://www.blooberry.com/indexdot/html/topics/urlencoding.htm). You can also use quoting functions in the python urllib module to URL-encode bytes (in particular, [urllib.parse.quote_from_bytes](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.quote_from_bytes), followed by ```.encode('ascii')``` to get the bytes from the string). In other cases, you may need to include binary values into your payload. The Python [struct](https://docs.python.org/3/library/struct.html) module can help you do that. For example, ```struct.pack("<Q", x)``` will produce an 8-byte (64-bit) binary encoding of the integer x.

Exercise 2.
----------------
Write an exploit that uses a buffer overflow to crash the web server (or one of the processes it creates). You do not need to inject code at this point. Verify that your exploit crashes the server by checking the last few lines of ```dmesg | tail```, using gdb, or observing that the web server crashes (i.e., it will print ```Child process 9999 terminated incorrectly, receiving signal 11```)

Provide the code for the exploit in a file called ```exploit-2.py```.

The vulnerability you found in Exercise 1 may be too hard to exploit. Feel free to find and exploit a different vulnerability.

You can check whether your exploits crash the server as follows:
```ps
student@CS431:~/lab$ make check-crash
```
Part 2: Code injection (4 points)
---------------------------------  

In this part, you will use your buffer overflow exploits to inject code into the web server. The goal of the injected code will be to unlink (remove) a sensitive file on the server, namely /home/student/grades.txt. Use ```zookd-exstack```, since it has an executable stack that makes it easier to inject code. The zookwsweb server should be started as follows.

```ps
student@CS431:~/lab$ ./clean-env.sh ./zookd-exstack 8080
```
You can build the exploit in two steps. First, write the shell code that unlinks the sensitive file, namely ```/home/student/grades.txt```. Second, embed the compiled shell code in an HTTP request that triggers the buffer overflow in the web server.

When writing shell code, it is often easier to use assembly language rather than higher-level languages, such as C. This is because the exploit usually needs fine control over the stack layout, register values and code size. The C compiler will generate additional function preludes and perform various optimizations, which makes the compiled binary code unpredictable.

We have provided shell code for you to use in /home/student/lab/shellcode.S, along with Makefile rules that produce ```/home/student/lab/shellcode```.bin, a compiled version of the shell code, when you run make. The provided shell code is intended to exploit setuid-root binaries, and thus it runs a shell. You will need to modify this shell code to instead ```unlink /home/student/grades.txt```.

To help you develop your shell code for this exercise, a program called run-shellcode is provided that will run your binary shell code, as if you correctly jumped to its starting point. For example, running it on the provided shell code will cause the program to ```execve("/bin/sh")```, thereby giving you another shell prompt:

```ps
student@CS431:~/lab$ ./run-shellcode shellcode.bin
```

Exercise 3.
----------------
Modify ```shellcode.S``` to unlink ```/home/student/grades.txt```. Your assembly code can either invoke the SYS_unlink system call, or call the unlink() library function.

To test whether the shell code does its job, run the following commands:
```ps
student@CS431:~/lab$ make
student@CS431:~/lab$ touch ~/grades.txt
student@CS431:~/lab$ ./run-shellcode shellcode.bin
# Make sure /home/student/grades.txt is gone
student@CS431:~/lab$ ls ~/grades.txt
ls: cannot access /home/student/grades.txt: No such file or directory
```
You may find [strace](https://linux.die.net/man/1/strace). useful when trying to figure out what system calls your shellcode is making. Much like with gdb, you attach strace to a running program:


```ps
student@CS431:~/lab$ strace -f -p $(pgrep zookd-)
```

It will then print all of the system calls that program makes. If your shell code isn't working, try looking for the system call you think your shell code should be executing (i.e., unlink), and see whether it has the right arguments.

Next, we construct a malicious HTTP request that injects the compiled byte code to the web server, and hijack the server's control flow to run the injected code. When developing an exploit, you will have to think about what values are on the stack, so that you can modify them accordingly.

When you're constructing an exploit, you will often need to know the addresses of specific stack locations, or specific functions, in a particular program. One approach to determine addresses is to use gdb. For example, suppose you want to know the stack address of the pn[] array in the http_serve function in zookd-exstack, and the address of its saved return pointer. You can obtain them using gdb by first starting the web server (remember clean-evn!), and then attaching gdb to it:

```ps
student@CS431:~/lab$ gdb -p $(pgrep zookd-)
...
(gdb) break http_serve
Breakpoint 1 at 0x5555555561c4: file http.c, line 275.
(gdb) continue
Continuing.
```

Be sure to run gdb from the ```~/lab directory```, so that it picks up the ```set follow-fork-mode child``` command from ```~/lab/.gdbinit```. Now you can issue an HTTP request to the web server, so that it triggers the breakpoint, and so that you can examine the stack of http_serve.

```ps
student@CS431:~/lab$ curl localhost:8080
```

This will cause gdb to hit the breakpoint you set and halt execution, and give you an opportunity to ask gdb for addresses you are interested in:

```ps
Thread 2.1 "zookd-exstack" hit Breakpoint 1, http_serve (fd=4, name=0x55555575fcec "/") at http.c:275
275         void (*handler)(int, const char *) = http_serve_none;
(gdb) print &pn
$1 = (char (*)[2048]) 0x7fffffffd4a0
(gdb) info frame
Stack level 0, frame at 0x7fffffffdcd0:
 rip = 0x5555555561c4 in http_serve (http.c:275); saved rip = 0x55555555587b
 called by frame at 0x7fffffffed00
 source language c.
 Arglist at 0x7fffffffdcc0, args: fd=4, name=0x55555575fcec "/"
 Locals at 0x7fffffffdcc0, Previous frame's sp is 0x7fffffffdcd0
 Saved registers:
  rbx at 0x7fffffffdcb8, rbp at 0x7fffffffdcc0, rip at 0x7fffffffdcc8
(gdb)
```

From this, you can tell that, at least for this invocation of http_serve, the pn[] buffer on the stack lives at address ```0x7fffffffd4a0```, and the saved value of ```%rip``` (the return address in other words) is at ```0x7fffffffdcc8```. If you want to see register contents, you can also use info registers.

Now it's your turn to develop an exploit.

Exercise 4.
----------------
Starting from one of your exploits from Exercise 2, construct an exploit that hijacks the control flow of the web server and unlinks ```/home/student/grades.txt```. Save this exploit in a file called ```exploit-4.py```.

Verify that your exploit works; you will need to re-create /home/student/grades.txt after each successful exploit run.

## Hints:

* First focus on obtaining control of the program counter.
* Sketch out the stack layout that you expect the program to have at the point when you overflow the buffer, and use gdb to verify that your overflow data ends up where you expect it to.
* Step through the execution of the function to the return instruction to make sure you can control what address the program returns to.
* The ```next```, ```stepi```, and [x](https://visualgdb.com/gdbreference/commands/x) commands in gdb should prove helpful.

Once you can reliably hijack the control flow of the program, find a suitable address that will contain the code you want to execute, and focus on placing the correct code at that address---e.g. a derivative of the provided shell code.

You can check whether your exploit works as follows:

```ps
student@CS431:~/lab$ make check-exstack
```

The test either prints "PASS" or "FAIL". We will grade your exploits in this way. Do not change the Makefile.

The standard C compiler used on Linux, gcc, implements a version of stack canaries (called SSP). You can explore whether GCC's version of stack canaries would or would not prevent a given vulnerability by using the SSP-enabled versions of zookd: zookd-withssp.

Part 3: Return-to-libc attacks (4 points)
-----------------------------------------
Many modern operating systems mark the stack non-executable in an attempt to make it more difficult to exploit buffer overflows. In this part, you will explore how this protection mechanism can be circumvented. Run the web server configured with binaries that have a non-executable stack, as follows.

```ps
student@CS431:~/lab$ ./clean-env.sh ./zookd-nxstack 8080
```
The key observation to exploiting buffer overflows with a non-executable stack is that you still control the program counter, after a ret instruction jumps to an address that you placed on the stack. Even though you cannot jump to the address of the overflowed buffer (it will not be executable), there's usually enough code in the vulnerable server's address space to perform the operation you want.

Thus, to bypass a non-executable stack, you need to first find the code you want to execute. This is often a function in the standard library, called libc, such as ```execve```, ```system```, or ```unlink```. Then, you need to arrange for the stack and registers to be in a state consistent with calling that function with the desired arguments. Finally, you need to arrange for the ret instruction to jump to the function you found in the first step. This attack is often called a return-to-libc attack.

One challenge with return-to-libc attacks is that you need to pass arguments to the libc function that you want to invoke. The x86-64 calling conventions make this a challenge because the first [6 arguments are passed in registers](https://eli.thegreenplace.net/2011/09/06/stack-frame-layout-on-x86-64). For example, the first argument must be in register %rdi (see man 2 syscall, which documents the calling convention). So, you need an instruction that loads the first argument into %rdi. In Exercise 3, you could have put that instruction in the buffer that your exploit overflows. But, in this part of the lab, the stack is marked non-executable, so executing the instruction would crash the server, but wouldn't execute the instruction.

The solution to this problem is to find a piece of code in the server that loads an address into %rdi. Such a piece of code is referred to as a "borrowed code chunk", or more generally as a [rop gadget](https://en.wikipedia.org/wiki/Return-oriented_programming), because it is a tool for return-oriented programming (rop). Luckily, zookd.c accidentally has a useful gadget: see the function accidentally.

Exercise 5.
-----------
Starting from your exploit in Exercises 2 and 4, construct an exploit that unlinks ```/home/student/grades.txt``` when run on the binaries that have a non-executable stack. Name this new exploit ```exploit-5.py```.

In this attack you are going to take control of the server over the network without injecting any code into the server. You should use a return-to-libc attack where you redirect control flow to code that already existed before your attack. The outline of the attack is to perform a buffer overflow that:

1) causes the argument to the chosen libc function to be on stack
2) then causes accidentally to run so that argument ends up in %rdi
3) and then causes accidentally to return to the chosen libc function

It will be helpful to draw a stack diagram like the figures in Smashing the Stack in the 21st CenturyLinks to an external site. at
(1) the point that the buffer overflows and
(2) at the point that accidentally runs.

You can test your exploits as follows:
```ps
student@CS431:~/lab$ make check-libc
```
Automated tools such as [ROPgadget.py](https://github.com/JonathanSalwan/ROPgadget) can assist you in searching for ROP gadgets, even finding gadgets that arise from misaligned parses. The VM already has ROPgadget installed.

You may find it useful to search for ROP gadgets not just in the zookd binary but in other libraries that zookd loads at runtime. To see these libraries, and the addresses at which they are loaded, you can run ( ulimit -s unlimited && setarch -R ldd zookd-nxstack ). The ulimit and setarch commands set up the same environment used by clean-env.sh, so that ldd prints the same addresses that will be used at runtime.

# 5. Finally ...
You are done! Submit your answers to the assignment by running ```make prepare-submit``` and upload the resulting ```lab1-handin.tar.gz``` file to the Canvas page. You can also submit partial solutions for Part 1 or Part 2 above. To submit for Part 1 and Part 2 of this assignment, run make ```prepare-submit-b``` and upload the resulting ```lab1b-handin.tar.gz``` file. To submit for Part 1, run make prepare-submit-a and upload the resulting ```lab1a-handin.tar.gz file.```