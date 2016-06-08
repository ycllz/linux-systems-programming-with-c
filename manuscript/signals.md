# signals

### Introduction
Signals are used to notify a process about some event. The event condition may be for ex: packet received on a network interface, packet sent on a network interface, watchdog timer expiry, floating point exception, invalid memory address read / write (segfault) etc. The linux provides 64 different signals range from SIGHUP to SIGRTMAX. The normal signals range from SIGHUP to SIGSYS and the real time signals start from SIGRTMIN to SIGRTMAX.

The command `kill -l` on the `bash` would give us the following.

```
 1) SIGHUP	        2) SIGINT	     3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	       7) SIGBUS	     8) SIGFPE	  9) SIGKILL   10) SIGUSR1
11) SIGSEGV	      12) SIGUSR2       13) SIGPIPE	14) SIGALRM   15) SIGTERM
16) SIGSTKFLT        17) SIGCHLD       18) SIGCONT	19) SIGSTOP   20) SIGTSTP
21) SIGTTIN          22) SIGTTOU       23) SIGURG     24) SIGXCPU   25) SIGXFSZ
26) SIGVTALRM        27) SIGPROF       28) SIGWINCH   29) SIGIO     30) SIGPWR
31) SIGSYS           34) SIGRTMIN      35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3
38) SIGRTMIN+4	   39) SIGRTMIN+5    40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8
43) SIGRTMIN+9	   44) SIGRTMIN+10   45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14	  49) SIGRTMIN+15   50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11      54) SIGRTMAX-10   55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6       59) SIGRTMAX-5    60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1       64) SIGRTMAX

```

The `SIGINT` and `SIGQUIT` are familiar to us as from the keyboard as we usually perform the `ctrl + c` (`SIGINT`) or `ctrl + \` (`SIGINT`) combination to stop a program.

Signals are also delivered to a process with the help of `kill` command. The manual page (`man  kill`) of kill command says that the default and easier version of kill command is the `kill pid`. Where `pid` is the process ID that is found via the `ps` command. The default signal is `SIGTERM (15)`. Alternatively a signal number is specified to the `kill` command such as `kill -2 1291` making a delivery of `SIGINT(2)` signal to the process ID 1291.


The linux also provide us a set of system calls API to use the signals that are delivered to the process. Some of the most important API are `sigaction` and `signal`. Lately `signalfd` is introduced that delivers the signals in a synchronous fashion to safely control the signals that occur at unknown or surprisingly.


### signal and sigaction

#### Signal

#### Sigaction

#### Key differences

A very good [stackoverflow question](http://stackoverflow.com/questions/231912/what-is-the-difference-between-sigaction-and-signal) here tells us why `sigaction` is better than `signal`. The key differences are:

1. the `signal()` does not necessarily block other signals from arriving while the current signal is being executed. Thus when more than one signal occur at the same time, it becomes more problematic to understand and perform actions. If it is on the same data, this might even get more complex. The `sigaction()` blocks the other signals while the handler is being executed.

2. As soon as the `signal()` handler is executed, the `signal()` sets the default handler to `SIG_DFL` which may be `SIGINT / SIGTERM / SIGQUIT` and the handler must reset the function back to itself so that when the signal occur again, the signal can be handled back. However, with the signal() allowing the handler to be called even though the handler is already executing, this will implicate a serious problem where in which the small window between the register and default would let the program go into `SIG_DFL`.


### signalfd

signalfd is a new system call introduced by the linux kernel. The signalfd system call returns a file descriptor associated with the signal. This file descriptor is then used to wait on the select, poll or epoll system calls.

The prototype of the `signalfd` is as follows.

`int signalfd(int fd, const sigset_t *mask, int flags);`

To use the `signalfd` one must include `<sys/signalfd.h>`.

If the fd argument is -1, the `signalfd` returns a new file descriptor. If fd argument is not -1, then the fd that is returned from previous calls to the `signalfd` must be given as an argument.

The mask argument is similar to the one that we pass to the `sigprocmask` system call. This allows the `signalfd` to create an `fd` out for the mask. As the mask is created, the signals in the mask should be blocked with the `sigprocmask`. This allows the correct functionality of the `signalfd`. 

The `flags` argument is usually set to 0. It is much similar to the O_NONBLOCK flag options of other system calls.

### sigaddset

### sigdelset

### sigemptyset

### sigprocmask