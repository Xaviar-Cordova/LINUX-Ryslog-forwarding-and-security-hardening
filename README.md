# LINUX-Ryslog-forwarding-and-security-hardening

Forwarding Cisco's equipments events and backups to Rsyslog server
![2024-10-07 15_58_38-RHEL Project Demo - GNS3](https://github.com/user-attachments/assets/6295cdff-010b-4a67-b100-80523355f28d)

In this example, we would like to emphasize security hardening for RHEL-based servers so we've created a scenario in which one was created as a source for network equipment to store their running-configs and forward their events logs. We'll be using GNS3 as our platform that'll include a Windows AD server, switch, and router to demonstrate proof of concept with Centos Stream 9 functioning as our Linux OS system.

we'll use nmtui to set a static IP 10.0.1.250, rename it to LXSR-01, and as we'll join to AD later set it to search for domain Xyz.com. Once the relationship has been established, we'll modify visudo to define admin rights for the AD security group Sudoers. As there will also be a regular AD group called Linuxuser to be granted non-privileged access to this server, we'll install Auditd to log security-related events such as login times. all services often need to be allowed to startup at boot, so for all future daemons let's assume we're running systemctl enable --now to accomplish this. once done we'll create an account called "localadmin" and add to the wheel group to take over privileges actions to leave the root alone for emergency purposes only.

The link is a Live 5-minute video [demonstration](https://github.com/user-attachments/assets/bdf604b2-da3b-4386-85f0-6ac569808f42)

![Screenshot 2024-11-25 155427](https://github.com/user-attachments/assets/d01b3810-b4b8-46de-99f6-7f5796337db1)

After we modified ryslog.conf file to begin listening on port UDP 514, we'll want to whitelist the port in firewall-cmd. Since this won't be a public-facing server and in a live environment would've been set on a separate VLAN than the native one, we'll set the zone to default to internal. it's a default port, but to be safe we'll also check Semange ports to see if the service is associated it. To verify, we should be using ss -lunp and tcpdump -i NIC-xyz UDP port 514 to ensure connectivity is occurring.
![2024-10-07 16_04_19-QEMU (LXSR-0-1) - TightVNC Viewer](https://github.com/user-attachments/assets/35d206ab-1b63-465c-bf24-aaa5c8e16c0d)

remote access to this servers will be useful as it not always feasible or even secure to have physical access to it considering by default rd.break can allow access if no grub password is set (as this a virtual platform, this portion can't be carried out). So Secure shell will prove vital to access it, but it too can benefit from enhanced security. we'll navigate to the sshd.conf file and modify idle time to close in 5 minutes, set the port to 2024, set maxauthtries to 3 to get logged quicker, and allowgroup to be sudoers and linuxusers.To acknowledge as passwords authentication is still enabled, ssh-keygen is considered best practice however we'll need to find a solutions that integrate with AD to allow users keys to attach to their profiles then to the computer themselves so passwords will be in used in this demonstration. we'll also need to whitelist the new port and modify the security context for 2024 to be associated to SSH.

![Fire-wall](https://github.com/user-attachments/assets/d1818e5a-6ee5-4481-99d6-52e1cc953d54)


For the files we intend to be stored here for network configurations, We'll be creating a public directory that hosts each branch's Topology layout and equipment running-configs. this may be one of the few area were execution right may be needed, but we do not wish for anyone to just have it. To ensure everyone can at least see directory files but can't execute we'll set umask o=rX. Anticipating future sub-directories will be created, Setfacl will be used to set the default rights to be assigned based on usergroups. One in particular that hosting the configurations, we'll remove any rights for linuxuser and only sudoers will have access.

![ACL](https://github.com/user-attachments/assets/6fd07eb4-74df-4ae2-8265-229908398b70)


To finalize our project on security and hardening, we'll sync our NTP to AD and disable unnecessary services. As mentioned the primary purpose of this server is to be a rsyslog server hosting network information, services like NFS or SAMBA should be disabled and blocked in our firewall, with semanage booleans checked if any additional features were turned on to accommodate them. It is also a good practice to modify over /etc/security/limit.conf to define the parameter in case DDOS does target it in later attacks.

![NTP](https://github.com/user-attachments/assets/facdb6e1-9f70-4ad0-8ca4-4abfd93255ae)
