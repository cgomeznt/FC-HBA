Como chequear una conexion HBA Fibre Channel
==============================================

Como chequear una conexion HBA Fibre Channel


Fibre Channel (FC) Host Bus Adapters(HBA) are interface cards that connects the host system to a fibre channel network or devices. The two major manufacturers of FC HBAs are QLogic and Emulex and the drivers for many HBAs are distributed in-box with the Operating Systems. If the drivers are not available on your Linux version, you need to install them manually and load the modules in kernel

Here is a step by step guide to verify that your FC HBAs installed and configured correctly.

Step-1: Determine the Manufacturer and Model of the HBAs.
Run the lspci command to list all PCI cards detected on the system.::

	mtbvir05:~$ sudo lspci -nn | grep -i fibre
	21:00.2 Fibre Channel [0c04]: Emulex Corporation OneConnect 10Gb FCoE Initiator (be3) [19a2:0714] (rev 01)
	21:00.3 Fibre Channel [0c04]: Emulex Corporation OneConnect 10Gb FCoE Initiator (be3) [19a2:0714] (rev 01)


The above output shows the system bus has detected two Emulex HBAs.

Step-2: Get the Vendor and Device IDs for the HBAs installed.
These can be obtained from the file "pci.ids".::

	mtbvir05:~$ find /usr/share/ -name pci.ids
	/usr/share/misc/pci.ids

	mtbvir05:~$ grep 19a2 /usr/share/misc/pci.ids
	19a2  Emulex Corporation

	mtbvir05:~$ grep 0714 /usr/share/misc/pci.ids
		0714  OneConnect 10Gb FCoE Initiator (be3)


The vendor id for Emulex is 19a2 and the device id is 0714.


Step-3: Check if the driver modules are installed.

This can be done by searching the list of available modules. (Replace 2.6.18-308.el5PAE with your kernel version in the command below).::

	mtbvir05:~$ grep 0714 /lib/modules/´uname -r´/modules.* | grep -i 19a2
	/lib/modules/3.2.0-4-amd64/modules.alias:alias pci:v000019A2d00000714sv*sd*bc*sc*i* lpfc

The above output shows that this HBA is supported by the module lpfc

Step-4: Check if the drivers for these HBAs are loaded in the kernel.

The lsmod command will list the currently loaded kernel modules.::

	mtbvir05:~$ lsmod | grep -i lpfc
	lpfc                  456734  40
	scsi_transport_fc      39180  1 lpfc
	scsi_mod              162321  8 scsi_tgt,hpsa,scsi_transport_fc,lpfc,sd_mod,sg,ses,scsi_dh


The output shows the module lpfc is loaded by the kernel. If you don't see any output for lsmod command then you can load the module using modprobe command.::

	# modprobe -v lpfc

Step-5: Getting detailed information

You can find detailed information about the fibre channel adapters in the location /sys/class/fc_host/.::

	mtbvir05:~$ ls -l /sys/class/fc_host/
	total 0
	lrwxrwxrwx 1 root root 0 Aug 20 23:34 host1 -> ../../devices/pci0000:20/0000:20:02.2/0000:21:00.2/host1/fc_host/host1
	lrwxrwxrwx 1 root root 0 Aug 20 23:34 host2 -> ../../devices/pci0000:20/0000:20:02.2/0000:21:00.3/host2/fc_host/host2

	mtbvir05:~$ ls -lL /sys/class/fc_host/
	total 0
	drwxr-xr-x 4 root root 0 Aug 20 23:23 host1
	drwxr-xr-x 4 root root 0 Aug 20 23:24 host2


The directories host1 and host2 in the example above contains information specific to each adapter like node name (WWN), port name (WWN), type, speed,state etc.,

An easier way to get this information is to use the systool command.::

	mtbvir05:~$ apt-cache search systool
	sysfsutils - sysfs query tool and boot-time setu

	mtbvir05:~$ sudo apt-get install sysfsutils

	mtbvir05:~$ systool -c fc_host
	Class = "fc_host"

	  Class Device = "host1"
		Device = "host1"

	  Class Device = "host2"
		Device = "host2"


The -v option gives you detailed output.::

	mtbvir05:~$ systool -c fc_host -v host1
	Class = "fc_host"

	  Class Device = "host1"
	  Class Device path = "/sys/devices/pci0000:20/0000:20:02.2/0000:21:00.2/host1/fc_host/host1"
		active_fc4s         = "0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 "
		dev_loss_tmo        = "30"
		fabric_name         = "0x100000110a02ff4d"
		issue_lip           = <store method only>
		max_npiv_vports     = "255"
		maxframe_size       = "2048 bytes"
		node_name           = "0x50060b0000c28a0d"
		npiv_vports_inuse   = "0"
		port_id             = "0x110700"
		port_name           = "0x50060b0000c28a0c"
		port_state          = "Online"
		port_type           = "NPort (fabric via point-to-point)"
		speed               = "10 Gbit"
		supported_classes   = "Class 3"
		supported_fc4s      = "0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x01 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 "
		supported_speeds    = "10 Gbit"
		symbolic_name       = "Emulex 554FLB FV10.2.340.19 DV8.3.27"
		tgtid_bind_type     = "wwpn (World Wide Port Name)"
		uevent              =
		vport_create        = <store method only>
		vport_delete        = <store method only>

		Device = "host1"
		Device path = "/sys/devices/pci0000:20/0000:20:02.2/0000:21:00.2/host1"
		  uevent              = "DEVTYPE=scsi_host"

To give the WWN.::

	mtbvir05:~$ systool -c fc_host -v host1 | grep -i port_name
		port_name           = "0x50060b0000c28a0c"

And now go to associete de with fdisk.::

	 
	mtbvir051# multipath -ll
	mpathd (360002ac0000000000000000b00008648) dm-116 3PARdata,VV
	size=500G features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:1:2 sdk 8:160 active ready running
	  |- 1:0:0:2 sdj 8:144 active ready running
	  |- 2:0:0:2 sdl 8:176 active ready running
	  ´- 2:0:1:2 sdm 8:192 active ready running
	xen-img2_r5 (360002ac0000000000000000900008648) dm-32 3PARdata,VV
	size=3.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:0:1 sdf 8:80  active ready running
	  |- 1:0:1:1 sdg 8:96  active ready running
	  |- 2:0:0:1 sdh 8:112 active ready running
	  ´- 2:0:1:1 sdi 8:128 active ready running
	xen-img1_r5 (360002ac0000000000000000700008648) dm-0 3PARdata,VV
	size=1.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:1:0 sdb 8:16  active ready running
	  |- 2:0:0:0 sdd 8:48  active ready running
	  |- 2:0:1:0 sde 8:64  active ready running
	  ´- 1:0:0:0 sdc 8:32  active ready running
	mpathh (360002ac0000000000000001000008648) dm-120 3PARdata,VV
	size=1.0T features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:0:3 sdn 8:208 active ready running
	  |- 1:0:1:3 sdo 8:224 active ready running
	  |- 2:0:0:3 sdp 8:240 active ready running
	  ´- 2:0:1:3 sdq 65:0  active ready running
	mpathg (360002ac0000000000000000f00008648) dm-118 3PARdata,VV
	size=10G features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:1:4 sds 65:32 active ready running
	  |- 1:0:0:4 sdr 65:16 active ready running
	  |- 2:0:0:4 sdt 65:48 active ready running
	  ´- 2:0:1:4 sdu 65:64 active ready running
::

	mtbvir05:~$ systool -c fc_transport -v

 

look where say mpath*, you identify this word in fdisk.::

	mtbvir05# fdisk -l 2>/dev/null  | grep mpathg
