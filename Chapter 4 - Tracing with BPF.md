# Introduction
- Tracing is a method to collect data for profiling and debugging.
- Using BPF for tracing means you can access almost any informatino from the kernel and your apps while adding a minimum amount of overhead to the system's performance and latency vs other tracing technologies.
- Using BPF also doesn't require developers to modify their apps for the only purpose of gathering data.
- BPF Compiler Collection (BCC) is a set of components that makes building BPF proograms more predictable by providing reusable components for common structures (e.g., Perf event maps), and integration with LLVM backend to provide better debugging.
- BCC also has bindings for different programming languages, which let you write user-space part of your BPF programs in a high-level language.

# Probes
- There are 4 main types of probes
    1. *Kernel probes*: These give you dynamic access to internal components in the kernel.
    2. *Tracepoints*: These provide static access to internal components in the kernel.
    3. *User-space probes*: These give you dynamic access to programs running in user-space.
    4. *User statically defined tracepoints*: These allow static access to programs running in user-space.

## Kernel Probes
- Let you set dynamic flags or breaks in almost any kernel instruction with minimum overhead.
- Kernel executes code attached to the probe when it reaches one of these flags, then resumes its usual routine.
- Kernel probes don't have a stable ABI, so they might change between versions!
- Divided into 2 categories: _kprobes_ and _kretprobes_

### Kprobes
- Let you insert BPF programs before any kernel instruction is executed, but you need to know the function signature that you want to break into.
