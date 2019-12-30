

### Important Notes

- Ensure to do the steps on Mint 19 or latest
- Git Checkout a Specific Branch `git checkout 2019.11br` then `make` it `https://github.com/opencomputeproject/onie/tree/2019.11br`
- HDD should be above VM
- Ensure to Power Off the VM when the first install on "ONIE Embed" is complete
- Make a Video

### FRR Links

https://www.2stacks.net/blog/getting-started-with-frr-on-eveng/#enable-eve-ng-telnet-access-to-frr-nodes


## Links

https://rendoaw.github.io/2019/02/Trying-Whitebox-OS-on-VM
https://esc.fnwi.uva.nl/thesis/centraal/files/f1729148638.pdf


https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/lastSuccessfulBuild/artifact/target/

### LastSUCCESSFULLBuild
https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/lastSuccessfulBuild/artifact/target/sonic-vs.bin

### LastSTABLEFullBuild
https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/lastStableBuild/artifact/target/sonic-vs.bin



https://sonic-jenkins.westus2.cloudapp.azure.com/job/vs/job/buildimage-vs-image/

https://learnvmware.online/2016/01/17/sdn-nos-and-white-box-switches/

http://lists.opencompute.org/pipermail/opencompute-onie/2016-March/001078.html

https://www.servethehome.com/get-started-with-40gbe-sdn-with-microsoft-azure-sonic-for-under-1k/#_copy_the_sonic_distribution_for_arista

https://ajketech.wordpress.com/2016/07/24/howto-install-ubuntu-on-an-onie-enabled-device-part-1/

https://wiki.akraino.org/display/AK/Edgecore+Switch+Usage+with+SONiC

https://community.mellanox.com/s/article/howto-enable-rdma-and-tcp-over-sonic--ocp-2017-demonstration-x#jive_content_id_ToR1_configuration

https://blog.mellanox.com/2019/10/open-ethernet-at-full-speed-with-leaf-spine-architecture-cumulus-and-sonic-vxlan-evpn-interoperate-over-mellanox/

https://github.com/Azure/SONiC/blob/master/doc/vrf/sonic-vrf-hld.md

https://academy.mellanox.com/en/wp-content/uploads/2019/08/MELLANOXS-SONIC-ADMINISTRATOR-TRAINING2.pdf?ls=pr&lsd=190925-SONiC-2

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



Image Build

Create empty file :  /opt/qemu/bin/qemu-img create -f qcow2  virtioa.qcow2 8G

1.
/opt/qemu-2.2.0/bin/qemu-system-x86_64 -nographic -drive file=virtioa.qcow2,if=virtio,bus=0,unit=0,cache=none -machine type=pc-1.0,accel=kvm -serial mon:stdio -nographic -nodefconfig -nodefaults -rtc base=utc -cdrom onie.iso -boot order=dc -m 3072

2. Embed ONIE

Now Stop ONIE booting and get to qemu prompt using key combo ctrl+a release keys and then press c, once appears (qemu) type quit.

3. Create New Lab - Ensure e1000 and connect all interfaces
4. Once DHCP , onie-nos-install sonic-vs - Configure per taste

5. STOP Lab
6. Commit Image
cd /opt/unetlab/tmp/0/72b1903b-4885-4a1c-b9e2-aed46032583a/1/
qemu-img commit virtioa.qcow2

7. Move the above file /opt/unetlab/addons/csr1000vng-sonic

/opt/unetlab/wrappers/unl_wrapper -a fixpermissions




### GRUB Changes

https://github.com/Azure/sonic-buildimage/blob/master/installer/x86_64/install.sh

```
# Add common configuration, like the timeout and serial console.
cat <<EOF > $grub_cfg
$GRUB_SERIAL_COMMAND
terminal_input serial
terminal_output serial
set timeout=5
```
### SONIC Mgmt Interface is `eth0` NOT Ethernet0

Notice the IP on `eth0`

```
Ethernet112            10.0.0.56/31         up/up         ARISTA13T0      10.0.0.57
Ethernet116            10.0.0.58/31         up/up         ARISTA14T0      10.0.0.59
Ethernet120            10.0.0.60/31         up/up         ARISTA15T0      10.0.0.61
Ethernet124            10.0.0.62/31         up/up         ARISTA16T0      10.0.0.63
Loopback0              10.1.0.1/32          up/up         N/A             N/A
docker0                240.127.1.1/24       up/down       N/A             N/A
eth0                   192.168.77.131/24    up/up         N/A             N/A
lo                     127.0.0.1/8          up/up         N/A             N/A
```

### Config Change and Applying Config without Reload

```
root@sonic:/home/admin# vi /etc/sonic/config_db.json
root@sonic:/home/admin# config reload
Clear current config and reload config from the file /etc/sonic/config_db.json? [y/N]: y
Stopping service swss ...
```
