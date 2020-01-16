# Systemd

Find the original slides here: 


https://www.redhat.com/files/summit/session-assets/2017/S103870-Demystifying-systemd.pdf 


[and the talk from RHEL summit 2018](https://www.youtube.com/watch?v=tY9GYsoxeLg)





### Ubuntu:

```bash 
Maintainer: /lib/systemd/system/
Administrator: /etc/systemd/system/
Non-persistent, runtime: /run/systemd/system
```

### RHEL:

```bash
Maintainer: /usr/lib/systemd/system/
Administrator: /etc/systemd/system/
Non-persistent, runtime: /run/systemd/system
```

Unit files in /etc/systemd/ take precedence over /usr/lib/systemd/ OR /lib/systemd/


Runlevels are now target units e.g:

   multi-user.target == runlevel3
 
 
   graphical.target  == runlevel5


You can glob services to work with multiple services


``` $ Systemctl {start,stop,restart,reload,enable,disable} httpd.service mariadb ```


(enable/disable is refered to as "on boot")

(When type isn't specified, it defaults to .service)

If logs are cut off, you can use -l


``` $ systemctl status nginx.service -l```


List loaded services:


``` $ systemctl -t service ```

List installed services:


``` $ systemctl list-unit-files -t service```

Check for services in failed state:

``` $ systemctl --state failed```


## Use Systemd Timers example (man systemd.timer):
```
  fstrim.timer
  [Unit]
  Description=Discard unused blocks once a week

  [Timer]
  OnStartUpSec=10min
  OnCalendar=weekly
  AccuracySec=1h
  Persistent=true
  [Install]
  WantedBy=multi-user.target
```


```
  fstrim.service
  [Unit]
  Description=Discard unused blocks

  [Service]
  type=oneshot
  ExecStart=/usr/sbin/fstrim /
```
 
## Customizing Units: Drop-ins

``` $ mkdir /etc/systemd/system/[name.type.d]/```

e.g: 


``` $ vim /etc/systemd/system/httpd.service.d/50-httpd.conf```

```
  [Service]
  Restart=always
  OOMScoreAdjust=-1000
```

``` $ systemctl daemon-reload```

OOMScoreAdjust:
Sets the adjustment level for the Out-Of-Memory killer for executed processes.
Takes an integer between -1000 (to disable OOM killing for this process) and 1000 (to make killing of this process under memory pressure very likely).

Want to see what's been altered on the system ?

``` $ systemd-delta```


## Systemd and Security

``` PrivateTmp=True ```

  File System namespace with /tmp and /var/tmp 


  (Files are under /tmp/systemd-private-*-[unit]-*/tmp)


``` PrivateNetwork=True ``` 


  Creates a network namespace with a single loopback device 


``` JoinsNamespaceOf= ```


  Enables multiple units to share PrivateTmp and PrivateNetwork


``` ProtectSystem=True ```


/usr & /boot are read-only 


if =full, /etc is also read-only


``` ProtectHome=True```


/home, /root, /run/user, will appear empty


Can be set to "read-only"


``` SELinuxContext= ```


Specify an SELinux context for the service 


``` NoNewPrivileges=True ```


  Ensure that a process & children cannot elevate privileges 
  
