---
title: "Post: CLion Remote Debug Using GDBServer"
last_modified_at: 2021-06-21T01:45:24+0900
categories:
  - Blog
tags:
  - debugging
---
When you want to debug with gdb remotely using CLion from Jetbrains, follow the steps below:

1. Run gdbserver on the target system
2. Create a GDB Remote Debug Configuration in CLion

To run gdbserver, execute the following command:
```shell
gdbserver :<port> /<path_to_executable>
```

If gdbserver is not installed, you can install it using your package manager by finding the gdb-gdbserver package.

In CLion, after creating the 'GDB Remote Debug' Configuration profile, enter the target IP and port where it says 'target remote' args: without any schema and run it.

If you encounter broken symbols or similar issues, ensure that the -DDEBUG option is set during compilation on the target. If it's set to -DNDEBUG, debug symbols are excluded, so change it to -DDEBUG. If the values are showing as optimized out, look for the -Ox option. 'x' denotes the level of build optimization, with numbers closer to 0 representing less optimization. If building with -O0 fails, try using the -Og option. It's said to optimize to a degree that does not interfere with debugging.