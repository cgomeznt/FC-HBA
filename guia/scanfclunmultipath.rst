
scan FC LUN multipath
=======================

Debe hacer que el kernel vuelva a cargar la configuaracion de la HBA, debe hacer esto primero.

 

Siempre es bueno ejecutar el siguiente comando para que haga un flush de las rutas de las HBA.::

	# multipath -F

Con el siguiente comando le decimos al kernel que busque nuevamente las HBA y con eso evitamos reiniciar el equipo.::

	 for host in ´ls /sys/class/fc_host´; do
		 echo "- - -" > /sys/class/scsi_host/${HOST}/scan
		 done

Con el siguiente comando refrescamos las tablas de rutas de las HBA.::

	# multipath

Ya aqui podemos ver la LUN asignada por storage y lo mas importante que debemos ver tal cual, el nombre de la WWN.::

	# multipath -ll | grep -i mpath


::

	root@mtbvir05# fdisk -l 2>/dev/null | grep mpath
	Disk /dev/mapper/mpathd: 536.9 GB, 536870912000 bytes

::

	root@mtbvir05# ls /sys/class/fc_host/
	host1  host2

::

	root@mtbvir05# tail -f /var/log/kern.log | grep -i link

::

	root@mtbvir05# multipath -ll
	mpathd (360002ac0000000000000000b00008648) dm-116 3PARdata,VV
	size=500G features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:1:2 sdk 8:160 active ready  running
	  |- 1:0:0:2 sdj 8:144 active ready  running
	  |- 2:0:0:2 sdl 8:176 active ready  running
	  ´- 2:0:1:2 sdm 8:192 active ready  running
	xen-img2_r5 (360002ac0000000000000000900008648) dm-32 3PARdata,VV
	size=3.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:0:1 sdf 8:80  active ready  running
	  |- 1:0:1:1 sdg 8:96  active ready  running
	  |- 2:0:0:1 sdh 8:112 active ready  running
	  ´- 2:0:1:1 sdi 8:128 active ready  running
	mpathk (30000000000000000) dm-131 3PARdata,VV
	size=10G features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=enabled
	  |- 1:0:1:4 sds 65:32 failed faulty running
	  |- 1:0:0:4 sdr 65:16 failed faulty running
	  |- 2:0:0:4 sdt 65:48 failed faulty running
	  ´- 2:0:1:4 sdu 65:64 failed faulty running
	mpathj (360002ac0000000000000001200008648) dm-120 3PARdata,VV
	size=1.0T features='0' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:0:3 sdn 8:208 active ready  running
	  |- 1:0:1:3 sdo 8:224 active ready  running
	  |- 2:0:0:3 sdp 8:240 active ready  running
	  ´- 2:0:1:3 sdq 65:0  active ready  running
	xen-img1_r5 (360002ac0000000000000000700008648) dm-0 3PARdata,VV
	size=1.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
	´-+- policy='round-robin 0' prio=0 status=active
	  |- 1:0:1:0 sdb 8:16  active ready  running
	  |- 2:0:0:0 sdd 8:48  active ready  running
	  |- 2:0:1:0 sde 8:64  active ready  running
	  ´- 1:0:0:0 sdc 8:32  active ready  running

::

	root@mtbvir05# echo "1" > /sys/class/fc_host/host1/issue_lip
	root@mtbvir05# echo "1" > /sys/class/fc_host/host2/issue_lip
	root@mtbvir05# echo "- - -" > /sys/class/scsi_host/host1/scan
	root@mtbvir05# echo "- - -" > /sys/class/scsi_host/host2/scan



root@mtbvir05# multipath
Aug 27 18:12:19 | failed to find rport_id for target0:0:0
Aug 27 18:12:19 | failed to find rport_id for target0:0:0


root@mtbvir05# multipath -ll
mpathd (360002ac0000000000000000b00008648) dm-116 3PARdata,VV
size=500G features='0' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:1:2 sdk 8:160 active ready  running
  |- 1:0:0:2 sdj 8:144 active ready  running
  |- 2:0:0:2 sdl 8:176 active ready  running
  ´- 2:0:1:2 sdm 8:192 active ready  running
xen-img2_r5 (360002ac0000000000000000900008648) dm-32 3PARdata,VV
size=3.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:0:1 sdf 8:80  active ready  running
  |- 1:0:1:1 sdg 8:96  active ready  running
  |- 2:0:0:1 sdh 8:112 active ready  running
  ´- 2:0:1:1 sdi 8:128 active ready  running
mpathk (30000000000000000) dm-131 3PARdata,VV
size=10G features='0' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=enabled
  |- 1:0:1:4 sds 65:32 failed faulty running
  |- 1:0:0:4 sdr 65:16 failed faulty running
  |- 2:0:0:4 sdt 65:48 failed faulty running
  ´- 2:0:1:4 sdu 65:64 failed faulty running
mpathj (360002ac0000000000000001200008648) dm-120 3PARdata,VV
size=1.0T features='0' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:0:3 sdn 8:208 active ready  running
  |- 1:0:1:3 sdo 8:224 active ready  running
  |- 2:0:0:3 sdp 8:240 active ready  running
  ´- 2:0:1:3 sdq 65:0  active ready  running
xen-img1_r5 (360002ac0000000000000000700008648) dm-0 3PARdata,VV
size=1.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:1:0 sdb 8:16  active ready  running
  |- 2:0:0:0 sdd 8:48  active ready  running
  |- 2:0:1:0 sde 8:64  active ready  running
  ´- 1:0:0:0 sdc 8:32  active ready  running
 


root@mtbvir05# multipath -F
Aug 27 18:13:11 | mpathd: map in use
Aug 27 18:13:11 | xen-img2_r5: map in use
Aug 27 18:13:11 | xen-img1_r5: map in use


root@mtbvir05# multipath -ll
mpathd (360002ac0000000000000000b00008648) dm-116 3PARdata,VV
size=500G features='0' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:1:2 sdk 8:160 active ready  running
  |- 1:0:0:2 sdj 8:144 active ready  running
  |- 2:0:0:2 sdl 8:176 active ready  running
  ´- 2:0:1:2 sdm 8:192 active ready  running
xen-img2_r5 (360002ac0000000000000000900008648) dm-32 3PARdata,VV
size=3.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:0:1 sdf 8:80  active ready  running
  |- 1:0:1:1 sdg 8:96  active ready  running
  |- 2:0:0:1 sdh 8:112 active ready  running
  ´- 2:0:1:1 sdi 8:128 active ready  running
xen-img1_r5 (360002ac0000000000000000700008648) dm-0 3PARdata,VV
size=1.0T features='1 queue_if_no_path' hwhandler='0' wp=rw
´-+- policy='round-robin 0' prio=0 status=active
  |- 1:0:1:0 sdb 8:16  active ready  running
  |- 2:0:0:0 sdd 8:48  active ready  running
  |- 2:0:1:0 sde 8:64  active ready  running
  ´- 1:0:0:0 sdc 8:32  active ready  running



root@mtbvir05# fdisk -l 2>/dev/null | grep mpath
Disk /dev/mapper/mpathd: 536.9 GB, 536870912000 bytes
Disk /dev/mapper/mpathj: 1099.5 GB, 1099511627776 bytes
