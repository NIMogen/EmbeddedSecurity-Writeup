# Advanced Programming in the Unix Environment

- Directory and file system basics
    
    Every process has a working directory that can be modified with chdir()
    
    File descriptors identify files accessed by a process. When an existing file is accessed or a new file is created, the kernel returns a descriptor.
    
    3 descriptors are opened by default: stdin, stdout, stderr, with the integer values 0, 1, 2, respectively.
    
    Major functions for process control:
    
    pid_t fork()
    
    Creates a new child process as a duplicate of the current (parent) process in a seperate memory region
    
    exec()
    
    Replaces the current process image with a new process image; basically meaning that it executes what you supply it.
    
    pid_t waitpid()
    
    Wait for a child process to stop or terminate
    
- Signals
    
    Signals notify a process that some condition has occurred.
    
    Sometimes signals have a default action that occurs when they are received.
    
    You could also supply a function to execute when a certain signal is received, specifying your own signal behavior.
    
    Pressing the interrupt key or dividing by 0 both emit signals.
    
- Chapter 3, File i/o
    
    Remember that a file descriptor is any non-negative integer.
    
    int open()
    
    int openat()
    
    Every file has a current file offset, which is a non-negative integer measuring the number of bytes from the beginning of a file.
    
    Read and write operations start at the current file offset and increment the offset by the number of bytes read or written.