### Note
Though I have written a guide to VMware gdb, but VMware Workstation is too huge compared to QEMU, so when you failed and VM crashed, you need to much more time to kill the process of VMware and restart the whole program in order to try again. You can try run a QEMU inside the VM, since it already supports nested virtual machines.

# VMware-gdb

VMware Workstation (and Fusion) has a nice feature which allows to debug the Linux kernel running inside the VM with `gdb` on the host. This is enabled by adding few lines to the VM’s configuration file:

```
 debugStub.listen.guest64 = "TRUE"                     # Enable listener for 64 bit guest
 debugStub.listen.guest64.remote = "TRUE"              # Allow remote connection (optional)
 debugStub.port.guest64 = "8864"                       # Listen on specified port NNN, e.g., 8864
 debugStub.hideBreakpoints= "TRUE"                     # Set hardware breakpoints -- limited by HW (optional)
```

The configuration file should be located in the virtual machine folder and called `*.vmx`. For mac users, the configuration file is packed inside `*.vmwarevm`. So you just right click it and select `Show Package Contents`.

Then, as mentioned in our debug lecture, we want to disable KASLR as it will make the debugging harder. This is done by booting the kernel with the nokaslr option.

In Ubuntu, open `/etc/default/grub`, find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` and append `nokaslr` at the end. Then update GRUB:

```
sudo update-grub
```

In Fedora, this command will append the args to the default kernel.

```
sudo grubby --args="nokaslr" --update-kernel=DEFAULT
```

Also kernel config need to set `CONFIG_DEBUG_INFO` and `CONFIG_GDB_SCRIPTS` to y to get debug symbols. 
Don't forget to compile and install the kernel, and then reboot the VM.

Next, we want to obtain debug symbols for the running kernel and its modules. Copy the following files to the host OS as we need to load them into gdb:

```
vmlinux // your compiled kernel
```

## Using GDB

Now that we have the debug symbols, we are ready to fire off `gdb` in the host OS:

```
$ gdb vmlinux

(gdb) set architecture i386:x86-64
(gdb) target remote localhost:8864
(gdb) c
```

For those who use windows, you could clone the virtual machine and use one as debugger to run gdb and compile the vmlinuz and module files. The debug port is open in the host OS, so you can use the `ssh -L 8864:127.0.0.1:8864 HostIP` ssh to create a tunnel.

The rest should be the same with QEMU, just check the Guest Lecture Recording: Kernel Debbugging by Yifei Zhu. 

Ref:

- https://communities.vmware.com/t5/VMware-Fusion-Discussions/Using-debugStub-to-debug-a-guest-linux-kernel/td-p/394906
- https://xakcop.com/post/vmw-kernel-debugging/
