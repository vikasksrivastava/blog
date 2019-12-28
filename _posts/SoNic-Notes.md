

### Important Notes

- Ensure to do the steps on Mint 19 or latest
- Git Checkout a Specific Branch `git checkout 2019.11br` then `make` it
- HDD should be above VM
- Ensure to Power Off the VM when the first install on "ONIE Embed" is complete
- Make a Video

### FRR Links

https://www.2stacks.net/blog/getting-started-with-frr-on-eveng/#enable-eve-ng-telnet-access-to-frr-nodes

https://rendoaw.github.io/2019/02/Trying-Whitebox-OS-on-VM
https://esc.fnwi.uva.nl/thesis/centraal/files/f1729148638.pdf


https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/lastSuccessfulBuild/artifact/target/

https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/lastSuccessfulBuild/artifact/target/sonic-vs.bin

https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/

https://learnvmware.online/2016/01/17/sdn-nos-and-white-box-switches/

http://lists.opencompute.org/pipermail/opencompute-onie/2016-March/001078.html

https://ajketech.wordpress.com/2016/07/24/howto-install-ubuntu-on-an-onie-enabled-device-part-1/


sudo apt-get install -f

https://stackoverflow.com/questions/31391585/onie-stg-command-not-found-and-error-127-in-ubuntu-terminal/36048254

sudo apt install qemu-kvm git stgit gperf bison flex autoconf texinfo gawk libtool libtool-bin libncurses5-dev libexpat1 libexpat1-dev python3 xorriso help2man -y



git config --global user.email "you@example.com"
git config --global user.name "Your Name"


NOTICE THE COMMAND BELOW

vagrant@vagrant:~/onie/build-config$ make MACHINE=kvm_x86_64 all recovery-iso
==== patching  Linux ====

https://github.com/Azure/SONiC/blob/master/doc/ocp/201903-SONIC/hackathon/Virtual%20Switch%20X%20Hackathon.pdf

https://macauleycheng.gitbooks.io/sonic/chapter1/configuration/configdb.html

https://groups.google.com/forum/#!search/config_db%7Csort:date

https://stackoverflow.com/questions/15880303/serial-port-connection-between-host-and-guest-with-virtualbox


----

I ran into a similar situation running a QNX guest using VirtualBox 5.0.10 on an Ubuntu 14.04 host.

My solution seems general enough to apply to the above-mentioned case.

I configured the guest VM in the same way that Kells1986 setup his VM1:

Under the "Serial Ports"/"Port1" tab:

check "Enable Serial Port"
set "Port Number" to "COM1"
set "IRQ" to "4"
set "I/O Port" to "0x3F8"
set "Port Mode" to "Host Pipe"
uncheck "Connect to existing pipe/socket"
set "Path/Address" to an accessible file-system path (e.g. "/home/safayet/vmSerialPipe")
According to the VirtualBox manual:

You can tell VirtualBox to connect the virtual serial port to a software pipe on the host. ... On a Mac, Linux or Solaris host, a local domain socket is used ... On Linux there are various tools which can connect to a local domain socket or create one in server mode. The most flexible tool is socat and is available as part of many distributions.

A domain socket is an IPC mechanism on UNIX systems similar to a pipe.

I connected to the "pipe" end of the virtual serial port on the Ubuntu host using the socat command:

socat - UNIX-CONNECT:/home/safayet/vmSerialPipe

---

scp sonic@192.168.1.22:/home/sonic/Desktop/sonic-vs.bin .




NIE-RECOVERY:/ # onie-nos-install sonic-vs.bin
onie-nos-install sonic-vs.bin
discover: Rescue mode detected. No discover stopped.
ONIE: Executing installer: sonic-vs.bin
Verifying image checksum ... OK.
Preparing image archive ... OK.
Installing SONiC in ONIE
ONIE Installer: platform: x86_64-vs-r0
onie_platform: x86_64-kvm_x86_64-r0
Partition #1 is in use.
Partition #2 is in use.
Partition #3 is available
Creating new SONiC-OS partition /dev/sda3 ...
Could not create partition 3 from 268288 to 67377151
Unable to set partition 3's name to 'SONiC-OS'!
Error encountered; not saving changes.
Warning: The first trial of creating partition failed, trying the largest aligned available block of sectors on the disk
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot.
The operation has completed successfully.
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 727736 4k blocks and 182160 inodes
Filesystem UUID: a19a80c2-6331-4780-a60d-fc6255594402
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
  Booting `ONIE: Rescue'
