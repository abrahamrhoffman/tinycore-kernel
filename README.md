# tinycore-kernel
A small set of tools and instructions to custom compile the tinycore kernel

## Rationale
Tinycore is super small (~10MB). But sometimes the sparsity isn't compatible with Enterprise servers. For example, CONFIG_X86_X2APIC is required to boot Tinycore on motherboards with high IRQ count. To enable this kernel parameter, a recompile is required.

## VM Setup
- Start a Tinycore VM in Virtualbox
- Ensure the ISO you select uses the same kernel version that you are attempting to compile
- For example, the 9.0 release of Corepure64 ISO uses the 4.14.10-tinycore64 kernel.
- ** Cross-compilation toolchains are available, but they often break and generally require too much maintenance.

## TinyCore Setup
1. `vi /usr/share/udhcpc/default.script`, add `dns="8.8.8.8"` to the top.
2. `sudo echo "nameserver 8.8.8.8" > resolv.conf`
3. `tce-load -wi openssh`
4. `sudo cp /usr/local/etc/ssh/sshd_config.orig /usr/local/etc/ssh/sshd_config`
5. `sudo /usr/local/etc/init.d/openssh start`
6. Change the `tc` user password: `passwd`
7. SSH to the ip address: `ifconfig -a | grep inet`
8. Now, inside the SSH session: `tce-load -wi git glibc_apps ncurses-dev compiletc coreutils bc bash perl5 advcomp mkisofs-tools cdrtools`
9. Change to root: `sudo su`
10. Ensure a compatible Terminal variable is set: `TERM=xterm`

## Manual Build
1. Download the tinycore patched kernel and kernel config:
```
wget https://distro.ibiblio.org/tinycorelinux/9.x/x86_64/release/src/kernel/config-4.14.10-tinycore64
wget https://distro.ibiblio.org/tinycorelinux/9.x/x86_64/release/src/kernel/linux-4.14.10-patched.txz
```
2. Unpack the Kernel `tar -Jxf linux-4.14.10-patched.txz -C /tmp/my-folder`
3. Move the kernel config `mv config-4.14.10-tinycore64 /tmp/my-folder/linux-4.14.10/.config`
4. `make oldconfig` and answer all questions, in case you have no clue on the answer just provide the default ones (i.e. just hit Return)
5. `make menuconfig` and make any changes you need to the configuration
6. `make bzImage` to build the kernel itself
7. `make modules` to build the loadable modules
8. `mkdir /tmp/my-folder/modules`
9. `make INSTALL_MOD_PATH=/tmp/my-folder/modules modules_install`
10. Copy the desired drivers and modules from `/tmp/my-folder/modules/` to the location of your choice: e.x. for ISO remaster
