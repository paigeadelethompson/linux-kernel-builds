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
- linux-firmware *this is typically a package made available perhaps with this name or another depending on your distribution but it can be manually installed from:* https://github.com/endlessm/linux-firmware and follow the directions for manual install. Be advised that kernel modules sometimes retrieve these firmware blobs from a directory in `/lib` when the module is loaded. Also, `mkinitrd` and `dracut` will include some of the firmwares in the ramdisk when it is buitl depending on your distribution's specific configuration and hardware requirements.

# Build steps
1. `git clone https://github.com/torvalds/linux.git /usr/src/linux-git`
2. `mv /usr/src/linux /usr/src/linux-old`
3. `ln -s /usr/src/linux-git /usr/src/linux`
4. `cd /usr/src/linux && make mrproper && make clean`
5. `make allmodconfig`
6. `cat .config | grep "=m" > allmodconfig`
7. `mv allmodconfig .config`
8. `zcat /proc/config.gz | grep -v "=m" | grep -v "#" | grep "."  >> .config` *If you don't have* `/proc/config.gz` *then many linux distributions provide the kernel configuration in the /boot directory.* In this case you can do: 
`cat /boot/config-<version>-name-of-config-file | grep -v "=m" | grep -v "#" | grep "."  >> .config`
9. `make olddefconfig`
10. `make -j<nprocs> all` *replace <nprocs> with just the number of processors/cores/threads that you have to speed up the build as much as possible.*


## Recap
- Starting from step 5 we generated a kernel build configuration that enabled all targets which could be built as a module to be enabled.
- Created a file that omitted everything except for the module-specific targets and overwrote the kernel build config with it.
- Concatenated only the build specific options and statically compiled options of the distribution's known working kernel config to the end of the kernel build config.
- Ensured that any symbols which were deprecated or changed from the appended build specific and statically compiled options are up to date as well as any statically compiled options required to build all of the modules specified to be built are enabled. This is the job of `makeolddefconfig` as well as removing duplicate symbols; the last option will be given preference.
  
# Install
If you don't want to install from a package such as an rpm or deb, here's a manual strategy:
- *Optional step, requires qemu* Test the kernel `qemu-system-x86_64 -nographic -kernel arch/x86_64/boot/bzImage -append "console=ttyS0,11520n8"`
- copy `arch/x86_64/boot/bzImage` to `/boot` and name it as similarly to your distribution's naming conventions as possible.
- run `mkinitrd` or `dracut` depending on what tooling is appropriate for your distribution.
- run `update-bootloader` or `grub2-update` depending on which linux distribution you are using.
- *Optional* specify the newly created ramdisk to Qemu with `-initrd` and test it if desired.

# Customizing the kernel
This can be done after build step #9, by running `make menuconfig` prior to running `make all`. You may want to just follow the steps first and see if there are any problems asides from the one's you will create if you change any build-specific or statically compiled options that you don't understand. After that, you can start again with step #4 and then try to make customizations after step #9.

# Secureboot / IMA / EVM
- I'm open to suggestions (I've never been able to get secure boot to work) but luckily it can be done after the kernel is compiled and a few additional tools are required:
- https://github.com/rhboot/pesign
- https://github.com/lcp/mokutil
- certutil / pk12util / openssl
  
# Don't panic 
Whatever problem you're having is not impossible to solve: 
- `console=ttyS0,115200n8 console=tty0` and get a serial cable, or use a VM's serial port, if you can't get video to troubleshoot
- Feel free to open an issue on this repository and I'll be glad to try and help you
