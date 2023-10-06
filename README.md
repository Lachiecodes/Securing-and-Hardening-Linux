# Securing and Hardening a Linux System
![Screenshot 2023-10-06 200219](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/06cb9404-5fad-4cf9-8eee-5bf444ca2dab)


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

![Screenshot 2023-09-28 220156](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/bbb49f06-7f09-43a1-bb93-a11ec15c3560)

## Securing the Boot Loader

**Boot Sequence Order:**
1. BIOS executes MBR (Master Boot Record).
2. MBR executes GRUB2.
3. GRUB2 loads the kernel.
4. The kernel executes systemd which intialises the system.

**Physical Checklist:**
1. Physical Security (Kensington lock, etc).
2. Set up a BIOS password.
3. Configure the system to boot automatically from the Linux partition.
4. Set up a password for GRUB.

**Why is it important?**
- If a hacker has physical access to the computer they could alter grub and boot the system into a special mode of operation called single user mode where root is logged in automatically without a password.
- The attacker can also use the grub editor interface to change its configuration or gather information about the system.
- If its a dual boot system, the attacker can select at boot time another operating system which will ignore access controls and file permissions.

**How to Secure the Boot Loader**
- To address these issues, we will protect the grub bootloader with a password.
- Note that people with physical access to the files via other methods that grub cannot prevent, such as booting with a USB stick.
- You should setup a BIOS password as well and configure the system to boot automatically from the partition on which Linux is installed.
- We have to generate a hash password using a specific command: `grub-mkpasswd-pbkdf2` then enter your chosen password.<br>
  
![Screenshot 2023-09-28 220754](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/cd16b44d-1f67-4d8f-b03c-ec5bfec16c52)<br>

- It will generate a hashed form of the password.
- You then need to edit the grub main configuration file and add the hash of the password.
- That way your actual password is not visible in the grub scripts and a possible hacker cannot see it.
- You can view the main configuration file of GRUB2 using: `cat /boot/grub/grub.cfg`.
- It is not modified directly by the user, but by certain GRUB2 updates such as `update-grub2`.
- In fact, grub uses a series of scripts to  update this file.
- These are located in ls `/etc/grub.d`.
- Its best to change a custom file such as `40_custom` because it will not be overwritten even with the GRUB package is overwritten.
- Copy your hashed password, starting at the `grub.pbkdf2.sha512…`
- Open the custom file with: `sudo nano /etc/grub.d/40_custom`.<br>

![Screenshot 2023-09-28 221033](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/bed0be90-5f76-4fee-9832-c55f766b2b9b)<br>

  
- Add the line: `set superusers=”root”`.
- And on the next line: `passwd_pbkdf2 root HASHED PASSWORD`.<br>
  
![Screenshot 2023-09-28 222617](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/f5ba6858-55f1-4a1b-9ea3-265e42430bca)<br>

- Save the file and quit.
- Run the command to update the GRUB2 configuration file: `update-grub2`.<br>

![Screenshot 2023-09-28 223733](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/bc7be092-bef3-4c7a-8b05-25687416fdd9)<br>

- Now restart the system and you should be prompted for a password at boot.<br>

![Screenshot 2023-10-06 200002](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/ed1d6621-b8aa-4708-8bd2-4edbe45b8def)

