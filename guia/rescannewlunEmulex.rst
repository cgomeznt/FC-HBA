How to rescan for new LUNs on a Emulex FC Controller
=====================================================

rescan for new LUNs on a Emulex Fibre Channel Controller:

List all HBAs.::

	root@debdev # ls -l /sys/class/fc_host
	lrwxrwxrwx 1 root root 0 Oct 24 12:22 host10 -> ../
	lrwxrwxrwx 1 root root 0 Oct 24 12:22 host11 -> ../
	lrwxrwxrwx 1 root root 0 Oct 24 12:22 host7 -> ../.
	lrwxrwxrwx 1 root root 0 Oct 24 12:22 host9 -> ../.




Start a rescan for new LUNs on a HBA(means real FC Ports not Adapters).::

	root@debdev # echo "1" > /sys/class/fc_host/host10/issue_lip
	root@debdev # echo "- - -" > "/sys/class/scsi_host/host10/scan"


Get some fibre channel information. The fabric a HBA is connected to.::

	root@debdev # cat /sys/class/fc_host/host10/fabric_name
	0x100056eb2a367958


The WWNN (World wide node name).::

	root@debdev # cat /sys/class/fc_host/host10/node_name
	0x20000060ca3badb8


The WWPN (World wide port name).::

	root@debdev # cat /sys/class/fc_host/host10/port_name
	0x10000060ca3badb8


Port speed.::

	root@debdev # cat /sys/class/fc_host/host10/speed
	8 Gbit



Current Porttype.::

	root@debdev # cat /sys/class/fc_host/host10/port_type
	NPort (fabric via point-to-point)


Port State.::

	root@debdev # cat /sys/class/fc_host/host10/port_state
	Online

You can also use the systool to query the sys filesystem.::

	root@debdev # systool -c fc_host -v



To list the discovered WWNN and WWPNs use.::

	root@debdev # systool -c fc_transport -v




