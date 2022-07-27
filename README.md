# Preface
This will compile a kernel that will use your current kernels configuration, but also ensure the following: 
- The newest possible version of the Linux kernel will be compiled
- ALL of the modules which can be compiled will be compiled; this is a lot more than what default distribution kernels have sometimes 
- Build specific options and statically compiled features that the current kernel you are using require to boot will be enabled 
- No new build specific options or statically compiled features that could potentitally cause the kernel not to boot will be enabled 
- Build step #10 ensures that all targets will be built; this includes `.deb`, `.rpm`, `.tgz`, docs, headers, etc will be made available at the end of a successful build.

# Requirements
You need at least the following packages installed: 
- git 
- build-essential (gcc, automake, autoconf) 
- libncurses development headers (package name depends on your distribution) 
- mkinitrd, dracut, or mkinitramfs (depends on your distribution)
- linux-firmware *this is typically a package made available perhaps with this name or another depending on your distribution but it can be manually installed from:* https://github.com/endlessm/linux-firmware

# Build steps
1. `git clone https://github.com/torvalds/linux.git /usr/src/linux-git`
2. `mv /usr/src/linux /usr/src/linux-old`
3. `ln -s /usr/src/linux-git /usr/src/linux`
4. `make mrproper && make clean`
5. `make allmodconfig`
6. `cat .config | grep "=m" > allmodconfig`
7. `mv allmodconfig .config`
8. `zcat /proc/config.gz | grep -v "=m" | grep -v "#" | grep "."  >> .config` *If you don't have* `/proc/config.gz` *then many linux distributions provide the kernel configuration in the /boot directory.* In this case you can do: 
`cat /boot/config-<version>-name-of-config-file | grep -v "=m" | grep -v "#" | grep "."  >> .config`
9. `make olddefconfig`
10. `make -j<nprocs> all` *replace <nprocs> with just the number of processors/cores/threads that you have to speed up the build as much as possible.*

# Install
If you don't want to install from a package, here's *a* manual way and the way that I often do it myself:
- copy `arch/x86_64/boot/bzImage` to `/boot` and name it as similarly to your distribution's naming conventions as possible.
- run `mkinitrd` or `dracut` depending on what tooling is appropriate for your distribution.
- run `update-bootloader` or `grub2-update` depending on which linux distribution you are using.

# Customizing the kernel
This can be done after build step #9, by running `make menuconfig` prior to running `make all`
