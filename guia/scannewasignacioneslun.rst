Buscar nuevas asignaciones de LUN
===================================


How to scan new FC LUNS and  SCSI disks in Redhat Linux without rebooting the server?  Most of the Linux beginners have wondering how to do this and this article will be for them.It may look like very simple as we perform this in daily operation to scan luns but system has many work to do in background when you execute storage scanning commands. Redhat says this type of scan can be distributive,since it can cause delays while I/O operation timeout and remove devices unexpectedly from OS.So perform this scan when really you want to scan the disks and LUNS.

Scanning FC-LUN’s in Redhat Linux
+++++++++++++++++++++++++++++++++++

1.First find out how many disks are visible in “fdisk -l” .::

	# fdisk -l 2>/dev/null | egrep ‘^Disk’ | egrep -v ‘dm-‘ | wc -l

2.Find out how many host bus adapter configured in the Linux box.you can use “systool -fc_host -v” to verify available FC in the system.::

	# ls /sys/class/fc_host
	host0  host1

In this case,you need to scan host0 & host1 HBA.

3.If the system virtual memory is too low ,then do not proceed further.If you have enough free virtual memory,then you can proceed with below command to scan new LUNS.::

	# echo “1” > /sys/class/fc_host/host0/issue_lip
	# echo “1” > /sys/class/fc_host/host1/issue_lip

Note: You need to monitor the “issue_lip” in /var/log/messages to determine when the scan will complete.This operation is an asynchronous operation.

4.Verify if the new LUN is visible or not by counting the available disks.::

	# fdisk -l 2>/dev/null | egrep ‘^Disk’ | egrep -v ‘dm-‘ | wc -l

If any new LUNS added ,then you can see more count is more then before scanning the LUNS.

Scanning SCSI DISKS in Redhat Linux
++++++++++++++++++++++++++++++++++++

1.Finding the existing disk from fdisk.::

	[root@mylinz1 ~]# fdisk -l |egrep ‘^Disk’ |egrep -v ‘dm-‘
	Disk /dev/sda: 21.5 GB, 21474836480 bytes

2.Find out how many SCSI controller configured.::

	[root@mylinz1 ~]# ls /sys/class/scsi_host/host
	host0 host1 host2

In this case,you need to scan host0,host1 & host2.

3.Scan the SCSI disks using below command.::

	[root@mylinz1 ~]# echo “- – -” > /sys/class/scsi_host/host0/scan
	[root@mylinz1 ~]# echo “- – -” > /sys/class/scsi_host/host1/scan
	[root@mylinz1 ~]# echo “- – -” > /sys/class/scsi_host/host2/scan

4.Verify if the new disks are visible or not.::

	mtbvir05:~$ sudo tail -f /var/log/kern.log | grep -i link

	[root@mylinz1 ~]# fdisk -l |egrep ‘^Disk’ |egrep -v ‘dm-‘
	Disk /dev/sda: 21.5 GB, 21474836480 bytes
	Disk /dev/sdb: 1073 MB, 1073741824 bytes
	Disk /dev/sdc: 1073 MB, 1073741824 bytes

From Redhat Linux 5.4 onwards, redhat introduced ”/usr/bin/rescan-scsi-bus.sh” script to scan all the SCSI bus and update the SCSI layer to reflect new devices.

But most of the time,script will not be able to scan new disks and you need go with echo command.






 What does the echo “1” to the issue_lip file do? – SCAN SCSI in Linux?
Question 1:  What will happen if we issue the below command ?.
 
	# echo “1” > /sys/class/fc_host/host/issue_lip
 
 
Answer : 
 
    This operation performs a Loop Initialization Protocol (LIP) and then scans the interconnect and causes the SCSI layer to be updated to reflect the devices currently on the bus. A LIP is, essentially, a bus reset,  and will cause device addition and removal. This procedure is necessary to configure a new SCSI target on a Fibre Channel interconnect. Bear in mind that issue_lip is an asynchronous operation.
 
    The command may complete before the entire scan has completed. You must monitor /var/log/messages to determine when it is done. The lpfc and qla2xxx drivers support issue_lip
 
 
Question2 : 
 
What will happen if we issue the below command ? what does “- – -” mean in the command?
 
# echo “- – -” > /sys/class/scsi_host/host0/scan
 
 
Answer: 
 
It means that you are echoing a wildcard value of “channel target and lun”, and the operating system will rescan the device path.

 

 

mtbvir05:~$ sudo tail -f /var/log/kern.log | grep -i link

Multipath will automatically add new native devices to multipathed
