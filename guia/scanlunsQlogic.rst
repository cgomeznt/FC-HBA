Scan for New/Removed Luns Linux with QLogic FiberChannel HBA
===============================================================

This is the process I use to scan the QLogic Fibre Channel bus for new or removed LUNs. It was tested on Oracle Enterprise Linux 5 connected to a Compellent SAN, but I don’t see any reason it won’t work with another SAN.

Initially I was using a script from QLogic called “ql-dynamic-tgt-lun-disc.sh” but was having problems. Turns out that script is only for use with the QLogic Kernel module that you can download from QLogic. If you use that script with the default qla2xxx module that comes with the Linux kernel, the system will get unstable and often hang completely! Consider yourself warned!

# This is the process to add/remove LUNs on the Qlogic FC HBAs in Linux
# This assumes you’re using the default qla2xxx module from Linux and NOT
# the version of the module provided by QLogic

#################
# Add a new LUN #
#################

# On the SAN: Create Volume and MAP to server

# On the server you need to scan the FC Bus[s]:

# This does the actual scan on the server – this will
scan any fiber channel links you have::


	for host in ´ls /sys/class/fc_host´; do
	echo "- - -" > /sys/class/scsi_host/${HOST}/scan
	done

# Re-scans for multipathing the new device
multipath
# NOTE: Sometimes I have to run “multipath -F && multipath”
# or it won’t recognize the new paths. I don’t know why :(

# Verify that it sees the new volume
# /dev/mapper alias should match the volume serial number/WWN
# as listed on the SAN.::

	ls -ltra /dev/mapper

# You may now mount your volume

##################
# Removing a LUN #
##################
::

	umount /path/to/volume

# Get the drives associated with the LUN [ie sdc, sdj etc].::

	multipath -l <mpathalias>

# Flush the multipath list.::

	multipath -F

# Verify that the above drives are no longer used by ANYTHING
# Do this for EACH drive.::

	lsof /dev/sdX    #Where X = drive designation above

# Delete the drives.
# Do this for EACH drive.::

	echo 1 > /sys/block/sdX/device/delete  #Where X = drive designation above

# Unmap the volume on the SAN

# Re-scan then list multipath to make sure it doesn’t find the volume
# you just removed.::

	multipath
	multipath -l

# You should be done!
