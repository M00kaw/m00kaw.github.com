### Linux Performance Analysis in 60,000 Milliseconds 


Source: 
https://medium.com/netflix-techblog/linux-performance-analysis-in-60-000-milliseconds-accc10403c55


modified by asb


### Prerequisite 


apt install sysstat


### === 1 ===

   $ w 
   
   13:51:14 up 3 days,  1:31,  1 user,  load average: 4.20, 2.60, 1.20
   
   USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
   
   m00kaw   pts/0    89.150.129.198   13:45    1.00s  0.05s  0.01s w

Look at "load average: 4.20, 2.60, 1.20"

 The 3 numbers are average load the last minute, the last 5 minutes and last 15 minutes.
 A load average of 1 means a single CPU system is loaded all the time while on a 4 CPU system it means it was idle 75% of the time.

 Brendan Gregg:"(..) the 1 minute value is much lower than the 15 minute value, then you might have logged in too late and missed the issue."

Notice who's logged in, and 'WHAT' they are doing.

High load could be caused by a user doing something


### === 2 ===

(example from Netflix:) 

   $ dmesg |tail -n 20
   
   [1880957.563150] perl invoked oom-killer: gfp_mask=0x280da, order=0, oom_score_adj=0
   [...]
   [1880957.563400] Out of memory: Kill process 18694 (perl) score 246 or sacrifice child
   [1880957.563408] Killed process 18694 (perl) total-vm:1972392kB, anon-rss:1953348kB, file-rss:0kB
   [2320864.954447] TCP: Possible SYN flooding on port 7001. Dropping request.  Check SNMP counters.

This views the last 20 system messages, if there are any. Look for errors that can cause performance issues. The example above includes the oom-killer, and TCP dropping a request.


(example from CubeIO:)
   $ dmesg -T |tail -n 20
   
   [Mon Dec  9 08:03:59 2019] TCP: out of memory -- consider tuning tcp_mem
   [Mon Dec  9 08:04:03 2019] TCP: out of memory -- consider tuning tcp_mem
   [Mon Dec  9 08:04:11 2019] TCP: out of memory -- consider tuning tcp_mem
   [Mon Dec  9 08:04:12 2019] TCP: out of memory -- consider tuning tcp_mem
   [Mon Dec  9 08:05:00 2019] TCP: out of memory -- consider tuning tcp_mem


### === 3 ===

   $ vmstat 1
   
   procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
    r  b    swpd   free   buff  cache    si   so    bi    bo   in   cs us sy id wa st
    0  0      0 2034616 115108 1043032    0    0     7    33    5    4  2  0 97  0  0
    0  0      0 2034872 115108 1043036    0    0     0     0  353  295  4  1 96  0  0
    0  0      0 2033880 115108 1043052    0    0     0    88  851  690  3  0 97  0  1
    0  0      0 2033632 115108 1043064    0    0     0     0  929  703  5  1 95  0  0
    0  0      0 2032020 115108 1043080    0    0     0     0  652  494  5  0 96  0  0
    0  0      0 2030808 115108 1043084    0    0     0     0  523  440  2  1 98  0  0
    0  0      0 2029912 115108 1043084    0    0     0   124  196  209  1  0 99  0  0


**Procs**

 r: The number of runnable processes (running or waiting for run time).

 "To interpret: an “r” value greater than the CPU count is saturation."

 **Memory**

 swpd: the amount of virtual memory used.
 
 free: the amount of idle memory.

 **Swap**
 
 si: Amount of memory swapped in from disk (/s).
 
 so: Amount of memory swapped to disk (/s).

 -> Should be 0, if not 0, you're out-of-memory

 **CPU**
 
 These are percentages of total CPU time.
 
 us: Time spent running non-kernel code.  (user time, including nice time)

 sy: Time spent running kernel code.  (system time)


 id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
 
 wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.

### === 4 ===

    $ mpstat -P ALL 1
    
    Linux 4.9.0-8-amd64 (aws-api-00.xxxxxx) 	12/09/2019 	_x86_64_	(2 CPU)
    
    01:14:58 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
    01:14:59 PM  all    1.01    0.00    0.50    0.00    0.00    0.00    0.00    0.00    0.00   98.49
    01:14:59 PM    0    1.01    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   98.99
    01:14:59 PM    1    1.01    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   98.99
    
    01:14:59 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
    01:15:00 PM  all    1.51    0.00    0.50    0.00    0.00    0.50    0.50    0.00    0.00   96.98
    01:15:00 PM    0    0.00    0.00    1.01    0.00    0.00    0.00    0.00    0.00    0.00   98.99
    01:15:00 PM    1    3.03    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   96.97
    
    01:15:00 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
    01:15:01 PM  all    4.57    0.00    0.51    0.00    0.00    0.00    0.00    0.00    0.00   94.92
    01:15:01 PM    0    4.08    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   95.92
    01:15:01 PM    1    5.05    0.00    2.02    0.00    0.00    0.00    0.00    0.00    0.00   92.93

 "This command prints CPU time breakdowns per CPU, which can be used to check for an imbalance.
 
 -> A single hot CPU can be evidence of a single-threaded application."


### === 5 ===

    $ pidstat 1
    Linux 4.9.0-8-amd64 (aws-api-00.xxxxxx) 	12/09/2019 	_x86_64_	(2 CPU)
    
    01:23:49 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
    01:23:50 PM  1002      7821    1.00    0.00    0.00    1.00     0  node /apps/api/
    01:23:50 PM  1002      7843    1.00    0.00    0.00    1.00     0  node /apps/api/
    01:23:50 PM     0     16566    1.00    0.00    0.00    1.00     0  pidstat
    01:23:50 PM     0     23960    1.00    0.00    0.00    1.00     1  filebeat
    01:23:50 PM  1002     28113   12.00    1.00    0.00   13.00     1  node /apps/Voic
    
  
"Pidstat is a little like top’s per-process summary, but prints a rolling summary instead of clearing the screen. 

This can be useful for watching patterns over time, and also recording what you saw (copy-n-paste) into a record of your investigation."


    $ iostat -xz 1  
   
    Linux 4.9.0-8-amd64 (aws-api-00.xxxxxx) 	12/09/2019 	_x86_64_	(2 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               2.26    0.00    0.38    0.10    0.07   97.19
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    xvda              0.00     7.84    0.30    3.04    14.60    66.00    48.27     0.01    2.44   10.16    1.68   0.60   0.20
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.50    0.00    0.50    0.50    0.00   98.51
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    xvda              0.00    18.00    0.00    5.00     0.00    92.00    36.80     0.00    0.80    0.00    0.80   0.80   0.40
    
# -x     Display extended statistics.

# -z     Tell iostat to omit output for any devices for which there was no activity during the sample period.


**r/s, w/s, rkB/s, wkB/s:** These are the delivered reads, writes, read Kbytes, and write Kbytes per second to the device. 

Use these for workload characterization. A performance problem may simply be due to an excessive load applied.

**%util:** Device utilization. This is really a busy percent, showing the time each second that the device was doing work. 

Values greater than 60% typically lead to poor performance (which should be seen in await), although it depends on the device. 

Values close to 100% usually indicate saturation.


### === 7 ===

    $ free -h
                  total       used        free      shared     buff/cache   available
    Mem:           3.9G        923M        1.6G         42M        1.3G        2.6G
    Swap:            0B          0B          0B

buffers: For the buffer cache, used for block device I/O.

cached: For the page cache, used by file systems.

### === 8 ===

    $ sar -n DEV 1
    Linux 4.15.0-65-generic (elastic-00.fullrate.dk) 	12/10/19 	_x86_64_	(8 CPU)
    
    13:44:23        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
    13:44:24       ens160   1824.00    786.00   7323.50   2525.08      0.00      0.00      0.00      0.60
    13:44:24           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
    
    13:44:24        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
    13:44:25       ens160   1784.00    785.00   7063.16   2228.46      0.00      0.00      0.00      0.58
    13:44:25           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
    ^C
    
    
    Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
    Average:       ens160   1804.00    785.50   7193.33   2376.77      0.00      0.00      0.00      0.59
    Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00


-n        Report network statistics.

With the DEV keyword, statistics from the network devices are reported.

%ifutil:

Utilization percentage of the network interface (max of both directions for full duplex)


### === 9 ===

    $ sar -n TCP,ETCP 1
    Linux 4.15.0-65-generic (elastic-00.fullrate.dk) 	12/10/19 	_x86_64_	(8 CPU)
    
    13:52:31     active/s passive/s    iseg/s    oseg/s
    13:52:32         0.00      0.00   1574.00   1935.00
    
    13:52:31     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
    13:52:32         0.00      0.00      0.00      0.00      0.00
    (...)
    13:52:36     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
    13:52:37         0.00      0.00      0.00      0.00      0.00
    ^C
    
    
    Average:     active/s passive/s    iseg/s    oseg/s
    Average:         0.00      0.17   1594.67   1973.67
    
    Average:     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
    Average:         0.00      0.00      0.00      0.00      0.00
    
"think of active as outbound, and passive as inbound, but this isn’t strictly true (e.g., consider a localhost to localhost connection)."

"Retransmits are a sign of a network or server issue; it may be an unreliable network (e.g., the public Internet), or it may be due a server being overloaded and dropping packets."

### === 10 === 

    $ htop

CTRL-s to pause screen and CTRL-q to continue 
