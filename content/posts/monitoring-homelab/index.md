+++
title = "Homelab"
date = "2026-01-18"
+++

Because "Homelab" != "Unreliable".

<!--more-->

# Introduction

"To monitor or not to monitor", said Niks shortly before fixing his broken DNS configuration on his Samba 4 Active Directory virtual machine after missing notifications for several hours on his devices.

Post mortem of the issue revealed a broken cron job that had pulled down an error HTML page instead of expected configuration file. I guess I could improve the cron job, but where's the fun in that when I can add another service on top of that?

Gentlemen and Gentleladies, It's Zabbix's turn to take the stage to fly [1]. 

## Monitoring Proxmox

What I care about:
- Health of ZFS
- Health of Disks.
- Health of Linux system.

So I monitor Linux:
- Memory Usage (Max RAM vs Used)
- CPU utilization and IOWAIT.
- Disk space available. Since I don't use thin-pools only monitoring root makes most sense to me, but I monitor all so I can check historic data at later dates.

I monitor ZFS/disk [2-3]:
- Zpool IOPS and Throughput.
- Zpool health and scrub status.
- There's more, like ZFS ARC and datasets, vdev errors, but health currently is the main theme.
For whatever reason SMART is giving me errors on Zabbix 7.4 discovery so this is left as #ToFixLater





[^1]: Some of you might understand the reference, to those who don't - it's a book about esports. Guy was epic at esports at early stage of esports but got cryo frozen as he got some disease that couldn't be cured at the time. Anyhow, he got unfrozen and then went into esports again to win a tournament. Zabbix pissed me off when I was first learning it due to it's simplicity so I kept it in a shelf until recently when I got to use it again.

[^2]: You can find the template for OpenZFS [here](https://github.com/NiksSkersts/zabbix_zfs-on-linux). Heed the warning "I hardly know what I am doing, please check the code before applying it in your environment."

[^3]: Further reading "[Understanding Storage Performance Metrics](https://klarasystems.com/articles/understanding-storage-performance-metrics/)", "[OpenZFS: Using zpool iostat to monitor pool performance and health](https://klarasystems.com/articles/openzfs-using-zpool-iostat-to-monitor-pool-perfomance-and-health/)"
