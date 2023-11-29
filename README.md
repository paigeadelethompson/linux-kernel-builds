# Linux Kernel Builds 
These builds should include *ALL* drivers, and if anything actually is missing please cut a ticket in the issues page. 

## Version
The source code tag is specified as an environment variable in the repository configuration. For a complete list of tags: https://github.com/torvalds/linux/tags

# Testing 

```
mkdir -p work/rd ; cd work/rd
wget <.tar.XZ distribution from releases page>
tar -xJvf linux-6.6.0-x86_64.tar.xz
cp boot/vmlinuz-6.6.0 ../
cd ..
```

Test with qemu:

```
qemu-system-x86_64               \
-nodefaults                      \
-nographic                       \
-M q35                           \
-m 8G                            \
-smp 2                           \
-serial mon:stdio                \
-kernel vmlinux-6.6.0            \
-append "earlyprintk=ttyS0 console=ttyS0"
```
