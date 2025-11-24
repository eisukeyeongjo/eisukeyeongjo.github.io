# 15. How to fix `network 'default' is not active` error when starting virtual machine

```
username@localhost:~> sudo virsh net-list --all
[sudo] password for root: 
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes

username@localhost:~> sudo virsh net-start default
Network default started

username@localhost:~> sudo virsh net-list --all
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   no          yes

ezquerro@localhost:~> sudo virsh net-autostart default
Network default marked as autostarted

username@localhost:~> sudo virsh net-list --all
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes

```

