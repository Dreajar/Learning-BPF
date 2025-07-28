# Introduction
- BPF maps are key/value stores that live in the kernel and can be accessed by any BPF programs that know about them.
- Programs that run in user-space can also access these maps by using file descriptors.
- You can store any type of data in a map, as long as you specify the data size correctly beforehand (NOT AWS lmao).
- The kernel treats keys & values as binary blobs.

# Creating BPF Maps
- We create BPF maps with the `bpf` syscall and setting the 1st argument in the call to `BPF_MAP_CREATE`. If the call fails, the kernel returns `-1`. The call returns the file descriptor identifier associated with the map.
- The second argument in the syscall is the configuration for the map:
```c
union bpf_attr {
  struct {
    __u32 map_type;     /* one of the values from bpf_map_type */
    __u32 key_size;     /* size of the keys, in bytes */
    __u32 value_size;   /* size of the values, in bytes */
    __u32 max_entries;  /* maximum number of entries in the map */
    __u32 map_flags;    /* flags to modify how we create the map */
  };
}
```
- The third argument is the size of this configuration attribute.

## Sample BPF map
```c
union bpf_attr my_map {
  .map_type = BPF_MAP_TYPE_HASH,
  .key_size = sizeof(int),
  .value_size = sizeof(int),
  .max_entries = 100,
  .map_flags = BPF_F_NO_PREALLOC,
};

int fd = bpf(BPF_MAP_CREATE, &my_map, sizeof(my_map));
```

- Calls fail for 3 reasons:
    - If 1 of the attributes is invalid, the kernel sets the `errno` variable to `EINVAL`.
    - If the user executing the operation doesn't have enough privileges, the kernel sets the `errno` variable to `EPERM`.
    - If there isn't enough memory to store the map, the kernel sets the `errno` variable to `ENOMEM`.
- We can update elements in a BPF map with the helper function `bpf_map_update_elem(map_ptr, key_ptr, value, update_flag)`
- We can read elements with the function `bpf_map_lookup_element(map_ptr, key_ptr, ptr_to_store_value)`.
- Note that the kernel method and user-space helper differ in the first element.
- We can remove elements with the function `bpf_map_delete_element(map_ptr, key_ptr)`.
- We can iterate over elements in a BPF Map with `bpf_map_get_next_key()`.
- We can lookup and delete elements at the same time with `bpf_map_lookup_and_delete_elem()`.

# Concurrent Access to Map Elements
- Many programs can access the same maps concurrently which introduces race conditions.
- BPF introduced spin locks to prevent them.
- There are 2 BPF helper functions: `bpf_spin_lock` and `bpf_spin_unlock`.

# Types of BPF Maps
- Linux documentation defines maps as generic data structures to store different types of data.
- Kernel developers have added many specialized data structures that are more efficient in specific uses cases.
- Read the documentation.

# The BPF Virtual FileSystem
- Since BPF maps are based onfile descriptors, the map and all info it has disappears when the descriptor is closed.
- Version 4.4 of the Linux kernel introduced 2 new syscalls (`BPF_PIN_FD` and `BPF_OBJ_GET`) to allow pinning and fetching maps and BPF programs from a virtual filesystem.
- Maps and BPF programs pinned to this filesystem will remain in memory after the program that created them terminates.
- The default dir where BPF expects to find this virtual FS is `/sys/fs/bpf`.
- Note that these objects can only be manipulated by the `bpf` syscall - you cannot open them with `open`.
