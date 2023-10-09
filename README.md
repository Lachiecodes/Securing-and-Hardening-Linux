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

## Locking or Disabling User Accounts
- There are several ways in which a user account can be locked or disabled.
- One way in which we can do this is to lock password authentication for the user. Note that this does not entirely lock the user out of the account - they will still be able to log in using SSH public key authentication if it’s set.
- Run the command: `sudo passwd -l user`.<br>

![Screenshot 2023-10-03 091451](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/3fe3046f-03eb-4e0e-be90-4ea23d96e566)<br>

- To verify if the account is locked or disabled run the command: `sudo passwd —status user`. If you see a capital L it means its locked, NP means there is no password, and P means there is a valid password.
- You can also check the hash of the password using: `sudo cat /etc/shadow`. The password will have an ! at the front of it, which is not a valid file hash.<br>

![Screenshot 2023-10-03 091523](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/c971557d-6df3-4070-9Givi9f2-b57316316925)<br>

- To unlock the user user account, use the command: `sudo passwd -u user`.<br>

![Screenshot 2023-10-03 091919](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/0eaad4c1-6cf0-497d-ae55-c52e7f3b20d6)<br>

- To completely disable an account use the command: `sudo usermod —expiredate 1 user`. This sets the users account expiry date to 1 day after the Unix epoch, which is 00:00:00 UTC on 1 January 1 1970.
- To check the expiration date run: `sudo chage -l user` and you should see the expiration date as the 02 January 1970.<br>

![Screenshot 2023-10-03 092218](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/d0566ddf-cbb3-4f8a-b81e-f5c1277c8725)<br>

- To reenable the account, you can use the command: `sudo usermod —expiredate "" user`.<br>

![Screenshot 2023-10-03 092329](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/4446984d-3349-4a8b-b677-9e6183b99c46)

## Giving Limited root Privileges
- In Linux, any administrative command that changes the system in any way should be run as root.
- The users that belong to the sudo group are allowed to run commands as root by using sudo before the command name.
- To see the users that belong to the sudo group run: `grep sudo /etc/group`.<br>

![Screenshot 2023-10-03 093050](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/9f180bba-b919-493a-8754-4b231fa3c729)<br>

- Initially, only the user that was created during system installation is allowed to run commands as sudo.
- The admin can add other users to the sudo group and they will be allowed to run commands as sudo aswell.
- The sudo command will request the password of the current user, not the root password.
- By running sudo su you can become root temporarily until you log out.
- One possible problem of this approach, is that such a user is allowed to run any commands as root. It’s not possible to restrict their access to some parts of the system or some commands.
- For example, if you want a user to be able to update a system without being able to change the configuration of the web server, the classical approach doesn’t allow that.
- We can give limited privilege access to users or groups and configure per command privilege access by editing the sudoers configuration file located at `/etc/sudoers`.
- Due to it being possible to break the system with improper syntax when editing this file, it is not recommended to edit the file directly with a normal text editor.
- Always use the visudo command instead, as it validates the syntax of the file before saving to ensure.
- By default, the command uses Vim as the text editor, but you can configure it using the command: `sudo update-alternatives —config editor`. I prefer to use nano.<br>

![Screenshot 2023-10-03 122115](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/1739844c-d5e4-4080-98ff-9fe3f9657d96)<br>

- For an example, I will make a new user called user1 and edit their privileges: `sudo useradd -d /home/user1 -m -s  /bin/bash user1`.
- For now, try running the command `sudo cat /etc/shadow` as user1.<br>

![Screenshot 2023-10-03 094628](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/0862f46c-7aa7-43cf-80bd-a00c11ef2049)<br>

- We get an error that user1 is not in the sudoers file and that the incident has been reported.
- Now, to edit the file run the command: `sudo visudo`. We are going to allow user1 to have root access to the commands `ls` and `cat`.
- In the # User privilege specification section, add the line: `user1 ALL=(root) /usr/bin/ls,/usr/bin/cat`.
- We need to provide the absolute file paths to the commands. These can be found using the command `which` for example `which cat`, `which ls` or `which apt`.

![Screenshot 2023-10-03 094907](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/b26131bc-9d0e-46c7-970b-3eb3e05197b6)<br>

- Try to run the command `sudo cat /etc/shadow` as user1. As you can see it works now.<br>

![Screenshot 2023-10-03 095013](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/3fcc07e4-e1e4-41b2-8f9d-d57c4c496b6b)<br>

- Try to also run the command `sudo apt update` as user1. You will find that you get an error.<br>

![Screenshot 2023-10-03 095048](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/90db6825-4da9-4dbd-8bc3-d29c992465bc)<br>

- You can also use `PASSWD:` and `NOPASSWD:` before the absolute paths to specify whether to prompt the user for a password when using these commands.
- You can also group users in the alias section of the configuration file to easily specify privileges for different departments in an organisation.
- For this we will need a second user account, so run the command: `sudo useradd -d /home/user2 -m -s  /bin/bash user2`.
- For example, in the #User alias specification section we will add the line: `User_Alias MYADMIN=user1,user2`.
- Then, in the #User privilege specification, add the line: `MYADMIN ALL=(root) /usr/bin/netstat,FILE`.<br>

![Screenshot 2023-10-03 122146](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/db4d5426-ab95-4a8c-93b3-4703ca319f6c)<br>

- Test out the commands on both accounts, it should be working.<br>

![Screenshot 2023-10-03 103520](https://github.com/Lachiecodes/Securing-and-Hardening-a-Linux-System/assets/138475757/e38a954b-8530-468a-a29d-31f62964f241)

- There, we have successfully set specific root privileges for different user accounts.
