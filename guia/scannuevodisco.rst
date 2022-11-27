Scan nuevo disco y asignarlo sin reiniciar
===========================================

Necesitamos un comando para rescan nuevos SCSI bus. Esto lo que hace es decirle al Kernel que nuevamente busque dispositivos del tipo SCSI.

Buscamos los controladores SCSI que se encuentren en el servidor::

  # find /sys -iname 'scan'

Me retorno lo siguiente::

  /sys/devices/pci0000:00/0000:00:10.0/host2/scsi_host/host2/scan
  /sys/devices/pci0000:00/0000:00:07.1/ata1/host0/scsi_host/host0/scan
  /sys/devices/pci0000:00/0000:00:07.1/ata2/host1/scsi_host/host1/scan
  
Ahora le paso los parametros para que el Kernel realice el rescan::

  # echo "- - -" > /sys/devices/pci0000:00/0000:00:10.0/host2/scsi_host/host2/scan
  # echo "- - -" > /sys/devices/pci0000:00/0000:00:07.1/ata1/host0/scsi_host/host0/scan
  # echo "- - -" > /sys/devices/pci0000:00/0000:00:07.1/ata2/host1/scsi_host/host1/scan

Hacemos el fdisk y ahora veremos el nuevo SCSI sin necesidad de reiniciar::

  # fdisk -l
