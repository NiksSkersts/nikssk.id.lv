+++
title = "Second take on design of my Homelab"
date = "2025-09-06"
+++

Last documentation update I had was on my WikiJS instance under lock and key. This time I am making it public.
<!--more-->

My concerns regarding opening my previous WikiJS instance was that:
1. I am helping threat actors with free information about my homelab - security concerns.
2. I would make others think that I am like Scrooge McDuck: swimming in money.
3. I would expose bad design decisions / questionable IT practices to my current or future employer.

```
!!!
For any current or future threat actor thinking about having a go at my homelab.
I just finished uni. There's no money to be made here. Dirt poor.
All my homelab equipment is second-hand or gifted by my IT buddies.
Off ye go, ye naughty person!
```

Jokes aside, my current Homelab layout:
![Homelab](/homelab.drawio.svg)

## Workflow:

![Repositories](/repositories.drawio.svg)

You might ask why I keep homelab repo on my Gitea instance instead of Github. Reason for that:
- It frequently changes and I am too lazy to press manual sync. Roles/Collections are not changed that frequently. \
  I am thinking of using Ansible to push updates to Github and then send an API call to Gitea to sync. Maybe just configure Git to push to two repos? Will see.
- I haven't had the time to clean out variables. Everything valuable is currently encrypted using ansible-vault, but I would still prefer them not be there in the first place.

A lot of it really boils down to time. I have been keeping myself busy enough not to have such a pet named "Time". Eventually this workflow will be improved or changed - currently it's good enough.

## Two servers:
- qianbei.nikssk.id.lv physical server hosted at home.
- tan.nikssk.id.lv virtual private server hosted by [Netcup](https://www.netcup.com/de) in Germany.

**Qianbei.nikssk.id.lv**:
- Docker hosts all my docker containers and reverse proxy. Containers and reverse proxy configuration is dealt with using Ansible:
  - [ansible-role-caddy](https://github.com/NiksSkersts/ansible-role-caddy)
  - [ansible-role-docker](https://github.com/NiksSkersts/ansible-role-docker)
  - More to be added. My main homelab repository sits on Gitea.
- All VMs are automated by Ansible:
  - [ansible-linux-host](https://github.com/NiksSkersts/ansible-linux-host)
  - More to be added. My main homelab repository sits on Gitea.
- Proxmox is also automated by Ansible:
  - [community.proxmox](https://github.com/NiksSkersts/community.proxmox)
- I used to keep PSQL seperate, but recently I have went for just keeping database container for each service. Currently, I have PBS backups, but eventually I hope to add something like I used to have with the old DB where I just pg_dumpall to a seperate disk.

**Tan.nikssk.id.lv**:
- All HTTP/HTTPS traffic goes through Cloudflare.
- There's not much interesting there. Unlike the environment at home which is used for various fun things, Tan is the most prod-like environment I have. Built for a purpose.

## Network:
My main routing and firewall for homelab is done by Mikrotik CCR1009.
It hosts my internal DNS, DHCP and two WG servers:
- wg-vpn for my phones and laptops.
- wg-tan for connection to TAN.
Default firewall policies with exceptions:
- Allow WG from Internet (wg-tan limited by source IP). \
  WG-VPN: can access reverse proxy via HTTP/HTTPS. Some access to SSH and PVE. Access to DNS and public Internet. \
  WG-TAN: TAN can connect to pbs.nikssk.id.lv (Proxmox Backup Server) for backups.
- NAT to Minecraft server on DMZ, block any traffic from DMZ to internal subnets.
- I try to keep the setup as simple as possible. If I had a spare box, I could consider deploying another FW behind router and use router as something that deals with forwarding traffic. \
    However, that seems like too much effort. There's not much to protect in LAN - wanna hack my lightbulbs? My phone doesn't even connect to my Wi-Fi. It goes through VPN. I guess there's my gaming pc.


## Update on 3rd point - 13.09.2025

> Jack Haverty (13:55 – 14:48) Exactly. And that actually turned out to be one of the principles that ARPA used in its decision making about what to do. I got to talk to a lot of the ARPA guys over the years. And one of them told me that they actually get negative points from the management if they don’t have enough failures. **Because if they don’t have enough failures, that means they’re not trying things that are advanced. So they have to try things that are not known to work yet.**
> 
> So a lot of the things that happened inside the internet world were things that were done not because we knew they would work, but because we didn’t know that they would not work. So that’s a little scary. [^1]

That being said, those folk mentioned in the quote were walking an untrodden path at the early Internet. My path is nicely tiled city road. Not comparable, but his words did stuck to me. On the brightside, if I fail to find a solution on my own - I can search online.

[^1]: [TNO038: Building Things That People Will Use – ARPANET History with Jack Haverty](https://packetpushers.net/podcasts/total-network-operations/tno038-building-things-that-people-will-use-arpanet-history-with-jack-haverty/)

