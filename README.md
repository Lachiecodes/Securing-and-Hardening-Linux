# Securing and Hardening a Linux System

## Introduction
In this project I demonstrate a number of different ways in which you can configure settings in Linux Ubuntu to increase the security of the operating system. Although Linux is relatively secure and virus free out of the box, if you want enterprise level security for business purposes then you will need to configure it yourself. Here is a list of important steps you can take to secure your Linux system:
- Ensure Physical Security.
- Disable Booting from external media devices.
- Boot Loader Protection.
- Keep the OS Updated (only from trusted sources).
- Check the installed packages and remove unnecessary services.
- Check for Open Ports and stop the unnecessary ones.
- Enforce Password Policy
- Audit Passwords using John The Ripper.
- Eliminate unused and well-known accounts that are not needed.
- Give users limited administrative access.
- Do not use the root account on a regular basis and do not allow direct root login.
- Set limits using the `ulimit` command to avoid DoS attacks such as launching a fork bomb.
- Implement File Monitoring (Host IDS - AIDE).
- Scan for Rootkits, Viruses and Malware (Rootkit Hunter, chkrootkit, ClamAV).
- Use Disk Encryption to protect your data. Don’t forget to encrypt your backups aswell.
- Secure every Network Service especially SSHd.
- Securing Your Linux System with a Firewall (Netfilter/Iptables).
- Monitor the firewall and its logs.
- Monitor your logs and search for suspicious activity (logwatch).
- Scan your servers using a VAS such as Nessus or OpenVAS.
- Make backups and test them.

## Securing the OpenSSH Server

- Configuring SSH settings by editing the sshd_config file located in /etc/ssh/ with the command `sudo nano /etc/ssh/sshd_config`.<br>

  ![Screenshot 2023-09-28 215038](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/7f02903a-a8e9-4bf0-82f3-2cfb3026d5c6)<br>

- Make sure to create a backup of the sshd file before you edit it.
- After editing the file you will have to restart the server so that the changes will come into affect.
- Change the default port of SSH from 22 to a random one of your own choosing will avoid you being seen by casual scans, but not targeted attacks: `Port 2278`.
- Due to most SSH attacks on the internet being automated, changing the default port is highly recommended.
- PermitRootLogin by default is set to prohibit-password but this still allows public key authentication and other such methods. It’s safer to disallow this one: `PermitRootLogin no`.
- Limit users SSH access by specifying the allowed users with: `AllowUsers user1 user2 user3`.
- Configure an idle timeout interval to automatically log out clients that have been inactive for a certain period of time: `ClientAliveInterval 300` and `ClientAliveCountMax 0`.
- Another good setting to configure is the `MaxAuthTries` and `MaxStartups`. <br>

![Screenshot 2023-09-28 220335](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/de7da639-1bcc-4ec3-92b4-7df6c562671d)<br>

- You can also restrict SSH access to only allow connections from certain IP addresses using `iptables` to specify the address: `iptables -A INPUT -p tcp —dport 2278 -s x.x.x.x -j ACCEPT` and drop all other incoming connections:`iptables -A INPUT -p tcp —dport 2278 -j DROP`.<br>

![Screenshot 2023-09-28 220156](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/0aeeb8b1-dd9b-4475-8589-6da0b47eb3f9)
