# Securing and Hardening a Linux System

![Screenshot 2023-10-06 200655](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/e25b7275-f2f1-465c-b9be-1d0e83f1060a)


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
- Set limits using the ulimit command to avoid DoS attacks such as launching a fork bomb.
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

![Screenshot 2023-10-06 200002](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/ba222826-baa8-46b7-9a3f-3bbe2ad1cde9)

## Enforcing Password Policy
- Password policy is just as good as the strength of the users passwords.
- Password policies are a set of rules which must be satisfied. They usually include password age, length and complexity, number of login failures, and whether reusing old passwords is permitted.
- Password aging and length settings are defined in the following file: `sudo nano /etc/login.defs`.<br>

![Screenshot 2023-09-29 091249](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/081a9e6b-d789-4ae6-87d9-2002ab5a54a2)<br>

- Note that all the parameters in login.defs are only applicable to new accounts and not existing ones.
- You can configure the maximum number of days a password can be used, the minimum number of days allowed between password changes and the number of days warning given before the password expires.
- Another password settings that is not showed in the file but exists is PASS_MIN_LENGTH. You can add it to the file and set it to a length greater than 12, which is an acceptable length.
- You can use the command `man login.defs` to check any other parameters that you may want to add to your password policy.
- To change the password expiry policy for existing accounts use the `chage` command.
- For example, to change the number of days between password change, use the command `sudo chage -M 60 user`.
- You can check that the settings have worked using `sudo chage -l user`.<br>

![Screenshot 2023-09-29 092556](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/72d318e2-b049-4ea5-9831-1528414b8311)<br>

- To enforce password complexity on most Linux distributions we use PAM, which means pluggable authentication module.
- The configuration file for pam can be found in `/etc/pam.d/common-password` on Ubuntu based systems.
- To use PAM on Ubuntu based systems, make sure the following package is installed using: `sudo apt install libpam-pwquality`.
- Open the common password file in your favourite text editor: `sudo nano /etc/pam.d/common-password`.<br>

![Screenshot 2023-09-29 094607](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/4e908792-bae7-4777-b2a3-7c95b288b95e)<br>

- Find the line with pam_pwquality.so and add the following attributes: `retry=3 minlen=12 difok=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1`.
- `retry=3` will prompt the user 3 times before exiting and returning an error.
- `minlen=12` specifies that the password can not be less than 12 characters.
- `difok=3` sets the minimum number of characters that must be different from the old password.
- `ucredit=-1` requires at least one uppercase character in the password.
- `lcredit=-1` requires at least one lowercase character in the password.
- `dcredit=-1` requires at least one numerical character in the password.
- `ocredit=-1` requires at least one special character in the password.<br>

![Screenshot 2023-09-29 094945](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/1c516974-3a3f-45e8-8fc5-7574409c197e)<br>

- Now that you have set these requirements, try to change your password to something that doesn’t fit and make sure that its all working!<br>

![Screenshot 2023-09-29 095338](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/14e495cc-8608-4412-ba45-271085d6c5f9)
