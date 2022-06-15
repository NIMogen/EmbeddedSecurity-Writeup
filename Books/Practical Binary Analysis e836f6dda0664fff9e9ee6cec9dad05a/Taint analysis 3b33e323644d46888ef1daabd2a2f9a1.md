# Taint analysis

Taint analysis is also called data flow tracking.

Taint sources - Taint sinks - Taint propagation

**Taint Sources**

Where you select the data that you want to track. 

If you wanted to check all data received from a network, you would use whatever DTA tool you are using to instrument a syscall like recv or recvfrom, and mark all bytes returned from those calls as tainted, so that they could be tracked. If you wanted to check input recieved from a file, you would choose read() instead.

Note that a taint source does not necessarily have to be a function or a syscall.