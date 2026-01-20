+++
title = "Homelab"
date = "2025-10-26"
+++

Here's my current Homelab layout and documentation.

<!--more-->

My concerns regarding publishing my homelab documentation:
1. I am helping threat actors with free information about my homelab - security concerns.
2. I would make others think that I am like Scrooge McDuck: swimming in money.
3. I would expose bad design decisions / questionable IT practices to my current or future employer.

My documentation will be written in Question => Answer format. It seems that this is the best way for my brain to parse information.

## Todo

!TODO:
- [ ] I would like OSPF capable router on tan.nikssk.id.lv for dynamic routing.
- [ ] Create a diagram for AD sync.
- [ ] Simplify GIT workflow.
- [ ] Make public:
  - [ ] Ansible code (without vars),
  - [ ] Scripts and anything like that,
  - [x] Redacted documentation.
- [ ] Find a way to securely store variables.
- [ ] Test Ansible Automation Platform instead of SemaphoreUI.
- [ ] Add sources.

## Homelab layout

1. qianbei.nikssk.id.lv is a physical server hosted at my home.
2. tan.nikssk.id.lv is virtual private server hosted by [Netcup](https://www.netcup.com/de) in Germany with Cloudflare providing proxy.

![Homelab](/homelab.drawio.svg)

## AD structure

![Homelab](/ad.drawio.svg)

```
DNS query flow

CLIENTS => Samba4 DNS => Unbound DNS => Public DNS over TLS.

```

Rsync setup.

```
! cat /etc/rsyncd.conf
[SysVol]
path = /var/lib/samba/sysvol/
comment = Samba Sysvol Share
uid = x
gid = x
read only = yes
auth users = sysvol-replication
secrets file = /path/to/rsyncd.secret

! crontab
*/30 * * * * rsync -XAavz --delete-after --password-file=/path/to/rsyncd.secret rsync://sysvol-replication@ip.ip.ip.ip/SysVol/ /var/lib/samba/sysvol/

```

### Joining client to AD
```
apt install realmd sssd oddjob oddjob-mkhomedir adcli samba-common packagekit sssd-tools
apt install chrony ntpsec-ntpdate
ntpdate -bu ad1.nikssk.id.lv
realm join --user=administrator ad1.nikssk.id.lv
rm -f /var/lib/sss/db/cache_nikssk.id.lv.ldb
pam-auth-update # Tick "create home folder on login"
```


## Workflow:

![Repositories](/repositories.drawio.svg)

A lot of it really boils down to time. I have been keeping myself busy enough not to have such a pet named "Time". Eventually this workflow will be improved or changed - currently it's good enough.


## Q&A

### AD

Q: Why Rsync for synchronising Sysvol? \
A: Because Samba4 does not provide a native way to sync Sysvol for GPOs. This is the recommended way to do it in official Samba documentation.

### Databases

Q: Centralized vs decentralized database. \
A: I used to keep PSQL separate, but recently I have went for just keeping database container for each service. Currently, I have PBS backups, but eventually I hope to add something like I used to have with the old DB where I just pg_dumpall to a separate disk.

### DNS, DHCP, and Network in general
Q: Why Unbound in the middle for DNS queries? \
A: For an easy way to fight modern data gathering measures of the Internet. ^_^
Samba4 DNS only resolves names withing the domain and the rest are forwarded.

Q: What provides DHCP? \
A: SRV/TAN network uses static networking as I deploy VMs using Ansible with these values set in variables. \
LAN uses DHCP provided by my Mikrotik router that points DNS/NTP to ADs.

### Storage

Q: Mirrors, mirrors everywhere? \
A: Yes. No RAIDZ. This decision was influenced by [JRS Systems: ZFS: You should use mirror vdevs, not RAIDZ.](https://jrs-s.net/2015/02/06/zfs-you-should-use-mirror-vdevs-not-raidz/) alongside research on the www. \
I have quite a lot of disk space available and not enough data. No need to squeeze as much space from those disks of mine, if I don't have a way to use it. Mirrors are easier to deal with, and according to the Internet - faster.

Q: Why bother with RAID? \
A: If I can avoid it, I can't be arsed to rebuild things when my hard drive crashes. Even if I have documentation and backups present, it would still take quite a while to get it back to working order. Also there's performance benefits in using RAID1 and RAID10. Debatable if a few users would ever notice it, however, the thought makes my heart tingle with joy.

Q: Backups on the same machine? \
A: I am very ashamed of it, but right now I don't have the means (read: money) to get another box just for backups. However, even if I did, I would still keep Proxmox Backup Server on the same server and instead put backup server somewhere offsite. \
I am working on replicating some of my most valuable data between tan.nikssk.id.lv and qianbei.nikssk.id.lv. Maybe with a help of central PSQL DB with replication. That would also enable me to have High Availability setup for things like Bitwarden.

### Thoughts

> Jack Haverty (13:55 – 14:48) Exactly. And that actually turned out to be one of the principles that ARPA used in its decision making about what to do. I got to talk to a lot of the ARPA guys over the years. And one of them told me that they actually get negative points from the management if they don’t have enough failures. **Because if they don’t have enough failures, that means they’re not trying things that are advanced. So they have to try things that are not known to work yet.**
> 
> So a lot of the things that happened inside the internet world were things that were done not because we knew they would work, but because we didn’t know that they would not work. So that’s a little scary. [^1]

That being said, those folk mentioned in the quote were walking an untrodden path at the early Internet. My path is nicely tiled city road. Not comparable, but his words did stuck to me. On the brightside, if I fail to find a solution on my own - I can search online.


## Sources

[^1]: [TNO038: Building Things That People Will Use – ARPANET History with Jack Haverty](https://packetpushers.net/podcasts/total-network-operations/tno038-building-things-that-people-will-use-arpanet-history-with-jack-haverty/)

