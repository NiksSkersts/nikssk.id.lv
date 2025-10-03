+++
title = "Domain NIKSSK.ID.LV"
date = "2025-09-20"
private = "true"
+++


<!--more-->

Samba4 is easy once you get hang of it. Until then.
![cat-cry.jpg](/static/memes/cat-cry.jpg)

---

Useful resources:
- [wiki.samba.org](https://wiki.samba.org/index.php/Main_Page)
- [samba.tranquil.it](https://samba.tranquil.it/doc/en/index.html)


I found both to be good sources of information, however, I would read the wiki.samba.org first.
I found samba.tranquil.it contradicts official wiki from time to time, but it is much easier to read than wiki.samba.org.


My implementation of NIKSSK.ID.LV:
![My domain](/static/domain.drawio.svg)

Nothing fancy, but I have:
- 2 ADs (which I read is a good practice),
- 2 FS/File storage VMs for Samba shares and Docker. \
  As per documentation, Samba doesn't like if you keep shares on AD.
- Any other VM.


Servers and physical devices are already domain joined. Mobile devices with Linux/Windows have Wireguard VPN.

I would like to extend the way I deploy VMs.
Currently I have cloud-init that provisions default user and network config.
I plan to keep this default user as "Break Glass" account with SSH key authentification, and domain join each of the VM for admin/service accounts and groups. 
I am currently doing this on fsx.nikssk.id.lv VMs where docker containers are set to use uid:gid of domain accounts.

Configuration of FS VMs using Ansible: \
--- After this all I need is to domain join the device. ---
```yaml
    - name: FS specific
      # FS are member servers!
      tags:
        - role-fs
      when: "'fs' in group_names"
      block:

        - name: Install packages
          ansible.builtin.apt:
            name:
              - acl
              - adcli
              - attr
              - samba
              - winbind 
              - libpam-winbind 
              - libnss-winbind 
              - krb5-config 
              - krb5-user 
              - dnsutils 
              - python3-setproctitle
            state: present

        # This is in form fsx.nikssk.id.lv - FQDN!
        # /etc/hostname
        - name: Set hostname
          ansible.builtin.hostname:
            name: '{{ inventory_hostname }}'

        # cloud-init overrides /etc/hosts on each boot. 
        # I am editing a template to replace localhost IP for FQDN for eth1 interface IP.
        - name: Override /etc/cloud/templates/hosts.debian.tmpl
          ansible.builtin.replace:
            path: /etc/cloud/templates/hosts.debian.tmpl
            regexp: '127\.0\.1\.1'
            replace: '{{ ansible_host }}'
        
        # I kept seeing various larger config examples online, but official docs say that this is enough.
        - name: Override /etc/krb5.conf 
          ansible.builtin.copy:
            dest: /etc/krb5.conf 
            content: |
              [libdefaults]
              	default_realm = {{ ad_realm }}
              	dns_lookup_realm = false
              	dns_lookup_kdc = true
        
        # For the life of me I couldn't get shares to work without \
        # winbind use default domain = yes
        # I liked seeing {{ ad_workgroup }}\{{ username }} instead of just {{ username }}.
        # Not sure why, yet.
        - name: Override /etc/samba/smb.conf
          ansible.builtin.copy:
            dest: /etc/samba/smb.conf
            content: |
              [global]
                workgroup = {{ ad_workgroup }}
                realm = {{ ad_realm }}
                security = ads
                log file = /var/log/samba/log.%m
                logging = file
                server role = member server
                # Interface config
                # Prevents Docker DNS entries :)
                bind interfaces only = Yes
                interfaces = lo eth1
                # IDMAP fallback
                idmap config * : backend = tdb
                idmap config * : range = 700001-800000
                # IDMAP AD
                idmap config {{ ad_workgroup }} : backend = ad
                idmap config {{ ad_workgroup }} : schema_mode = rfc2307
                idmap config {{ ad_workgroup }} : unix_nss_info = yes
                idmap config {{ ad_workgroup }} : range = 500-20000
                idmap config {{ ad_workgroup }} : default = yes
                # Kerberos and keytab
                sync machine password to keytab
                # Winbind
                winbind use default domain = yes
                winbind enum users = yes
                winbind enum groups = yes
                winbind refresh tickets = yes
                winbind offline logon = yes
                winbind nss info = rfc2307

                vfs objects = acl_xattr
                map acl inherit = Yes
                template homedir = /home/%U

                printcap name = /dev/null
                load printers = no
                disable spoolss = yes
                printing = bsd

                {{ ad_smb_shares }}
```
If you plan to copy+paste this for your own config, please be mindful that I am using idmap_ad as backend.

