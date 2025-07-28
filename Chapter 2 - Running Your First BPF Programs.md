# Prerequisites
- The BPF VM can run instructinos in response to events triggered by the kernel, but not all BPF programs have access to all events triggered by the kernel
- When you load a program into the BPF VM, you need to pick which type of program you're running. This informs the kernel about where your program will be triggered. It also tells the verifier which helpers are allowed in your program.

## Hello World
```c
#include <linux/bpf.h>
#define SEC(NAME) __attribute__((section(NAME), used))

SEC("tracepoint/syscalls/sys_enter_execve")
int bpf_prog(void *ctx) {
  char msg[] = "Hello, BPF World!";
  bpf_trace_printk(msg, sizeof(msg));
  return 0;
}

char _license[] SEC("license") = "GPL";
```

- Attribute SEC informs BPF VM when we want to run this program (when tracepoint in `execve` system call is detected).
- Here, we will see the message `Hello, BPF World!` every time the kernel detects a program executing another program.
- Since the Linux kernel is licensed under GPL, it can load only programs licensed as GPL; othrewise, the kernel will refuse to load our program :)

# BPF Program Types
- 2 main categories: tracing and networking
- Tracing:
    - Gives direct info about system behavior and hardware it's running on
    - Can access memory reginos related to specific programs and extract execution traces from running processes
    - Give direct access to resources allocated for each specific process, e.g., file descriptors, CPUs, memory usage etc.
- Networking
    - Inspect/manipulate network traffic in system
    - Let you filter packets coming from the network interface

## Socket Filter Programs
- `BPF_PROG_TYPE-SOCKET_FILTER` was the 1st program type to be added to the kernel.
- When you attach a BPF program to a raw socket, you get access to all packets processed by that socket.
- Socket filter programs give you access for observability only - you can't modify/change destination of packets.

## Kprobe Programs
- Functions you can attach dynamically to certain call points in the kernel.
- Let you use BPF programs as kprobe handlers.
- Really helpful for tracing.

## Tracepoint Programs
- Let you attach BPF programs to the tracepoint handler provided by the kernel.
- Defined with the point `BPF_PROG_TYPE_TRACEPOINT`
- Tracepoints are static marks in the kernel's codebase that let you inject arbitrary code; less flexible than kprobes since they need to be defined by the kernel befoehand but much more predictable.

## XDP Programs
- Let you write code that's executed very early on when network packet arrives at kernel.
- Defined with type `BP_PROG_TYPE_XDP`
- Exposes limited info from packet given that the kernel hasn't processed it much.
- Can `XDP_PASS` to pass the packet the next subsystem in the kernel; `XDP_DROP` to drop the packet; `XDP_TX` to forward the packet back to the NIC that recevied the packet.
- XDP lets you implement programs to protect your network against DDoS attacks.

## Perf Event Programs
- Let you attach your code to _Perf Events_.
- Defined with type `BPF_PROG_TYPE_PERF_EVENT`.
- Perf is an internal profiler in the kernel that emits performance data events for hardware and software. Can use it to monitor many things, e.g., CPU, software etc.
- When you attach a BPF program to Perf events, your code is executed every time Perf generates data for you to analyze.

## Cgroup Socket Programs
- Let you attach BPF logic to control groups (which contain present and future processes).
- Useful in container environments where groups of processes are constrained by cgroups.
- Cilium (LB & security) module for Kubernetes uses cgroup socket programs extensively.

## Socket Option Programs



## Socket Map Programs


## Cgroup Device Programs


## Socket Message Delivery Programs



## Raw Tracepoint Programs



## Cgroup Socket Address programs



## Socket Reuseport Programs



## Flow Dissection Programs



## Other BPF Programs


# The BPF Verifier
- Verifier first creates a DAG with the code. Each instruction becomes a node, and each node is linked to its next instruction.
- Verifier then performs DFS to ensure the program finishes.
- Max number of instructions to execute is 4096, but we can nest different BPF programs to work around the limitation.
- Program doesn't include any unreachable instructions (i.e., dead code).
- Verifier then performs a dry run of the BPF program and checks it doesn't execute any invalid instructions.
- Verifier also checks that all memory pointers are accessed and dereferenced correctly
- Finally, the dry run tells the verifier about the control flow and ensures all control paths lead to the `BPF_EXIT` instruction.

# BPF Tail Calls
- Kernel versions <5.2 have a hard limit of 4096 machine instructions.
- From kernel vesion 5.2, the instruction limit increases to 1 million. Tail call nesting is limited to 32 calls, i.e., you can have 32 programs in a chain.
- When you call a BPF program from another BPF program, the kernel resets the program context completely, so you need BPF maps to share info between programs.
