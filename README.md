# Born_to_be_root
Unix-administration project. Goal - get understanding unix systems

Project was developed in virtual machine 30Gb. Operating system choosed Debian.

Below command log of my project:

su
apt install sudo
vi /etc/ssh/sshd_config
	Port 4242
	PermitRootLogin no#prohibit-password
su -
visudo /etc/sudoers 
#copy to /etc/sudoers.d/my_sudoers
vi /etc/sudoers.d/my_sudoers 
#replace root to coskelet, delete another strings
chmod 0440  /etc/sudoers.d/sudoers_local

#port forvarding Setting -> Network -> Advanced -> Port forwarding
# valued ports numbers only: 4242

#in iTerm on host-machine:
ssh coskelet@localhost -p 4242
sudo usermod -a -G sudo coskelet

#config /etc/sudoers.d/my_sudoers:
#Defaults badpass_message=
#Defaults passwd_tries=

#via sudo or su -  mode:
apt install ufw
ufw allow 4242/tcp
ufw enable
ufw status numbered

#add groups
addgroup user42
#to check getent group user42 or:
getent group | grep coskelet

#user password rules
apt install libpam-pwquality

#monitor scripting
sudo apt-get update -y
sudo apt-get install -y net-tools
sudo vi /usr/local/bin/monitor.sh
sudo chmod +x /usr/local/bin/monitor.sh
#Add sudo priveleges to script in file sudoers:
sudo chmod 0660 /etc/sudoers.d/sudoers_local
vi /etc/sudoers.d/my_sudoers #mod: coskelet ALL(ALL) NOPASSWD:  /usr/local/bin/monitor.sh
sudo reboot
sudo crontab -u root -e
#Add: */10 * * * * /usr/local/bin/monitor.sh

#Script to remote-machine monitor
#
#!/bin/bash
wall $'#Architecture: ' `hostnamectl | grep "Kernel" | cut -d ' ' -f14` `cat /etc/hostname` \
        `hostnamectl | grep "Kernel" | cut -d ' ' -f15` \
        '#1 SMP' `dmesg | grep "#1 SMP" | awk -F '#1 SMP' '{print $2}'` \
        `hostnamectl | grep "Architecture" | cut -d' ' -f8` \
        `hostnamectl | grep "Operating System" | cut -d' ' -f6` \
$'\n#CPU physical : ' `cat /proc/cpuinfo | grep processor | wc -l` \
$'\n#vCPU :  ' `cat /proc/cpuinfo | grep processor | wc -l` \
$'\n'`free -m | grep "Mem" | awk '{printf "#Memory Usage: %s/%sMB (%.2f%%)", $3,$2,$3*100/$2 }'` \
$'\n'`df --total | grep "total" | awk '{printf "#Disk Usage: %d", $3/1024}'` \
`df --total -h | grep "total" | awk '{printf "/%dGB (%s)", $2,$5}'` \
$'\n'`top -bin1 | grep "%Cpu" | awk '{print "#CPU Load: ", $2 + $4 + $6}'`'%' \
$'\n#Last boot: ' `who -b | awk '{print $3" "$4" "$5}'` \
$'\n#LVM use: ' `lsblk | grep -m1 lvm | awk '{if ($6) {print "yes";exit;}else{print "no";exit}}'` \
$'\n#Connection TCP:' `netstat -an | grep ESTABLISHED |  wc -l` ' ESTABLISHED' \
$'\n#User log: ' `who | cut -d " " -f 1 | sort -u | wc -l` \
$'\nNetwork: IP ' `hostname -I`"("`ifconfig | grep ether | cut -d' ' -f10`")" \
$'\n#Sudo:  ' `grep -e 'sudo:.*COMMAND' /var/log/auth.log | wc -l`

#/etc/sudoers.d/sudoers_local:
#
# sudo pass
Defaults        badpass_message="what are you doing, n&gga?"
Defaults        passwd_tries=3
# sudo logs
Defaults        logfile="/var/log/sudo/sudo.log"
Defaults        log_input, log_output
# sudo settings
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
# sudo users
coskelet ALL=(ALL) NOPASSWD: /usr/local/bin/monitor.sh

#check PATH
echo 'echo $PATH' | sh
echo 'echo $PATH' | sudo sh

#make sftp server
sudo addgroup sftp_users
sudo useradd -m sftp_user -g sftp_users
sudo passwd sftp_user
sudo chmod 700 /home/sftp_user
sudo vim /etc/ssh/sshd_config

# SFTP config
Match group 		sftp_users
ChrootDirectory		/home
X11Forwarding 		no
AllowTcpForwarding	no
ForceCommand		internal-sftp

systemctl restart sshd
sftp -P4242 sftp_user@localhost
cd sftp_user


# Password strong policy
sudo apt-get install libpam-pwquality
sudo man pam_pwquality
sudo vim /etc/pam.d/common-password

# here are the per-package modules (the "Primary" block)
password	requisite			pam_pwquality.so retry=3 \
	minlen=10 ucredit=-1 dcredit=-1 lcredit=-1 maxrepeat=3 usercheck=1 difok=7 enforce_for_root
password	[success=1 default=ignore]	pam_unix.so obscure use_authtok try_first_pass yescrypt
# here's the fallback if no module succeeds
password	requisite			pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
password	required			pam_permit.so
# and here are more per-package modules (the "Additional" block)
# end of pam-auth-update config

sudo vim /etc/login.defs

# Password aging controls:
#
#	PASS_MAX_DAYS	Maximum number of days a password may be used.
#	PASS_MIN_DAYS	Minimum number of days allowed between password changes.
#	PASS_WARN_AGE	Number of days warning given before a password expires.
#
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARN_AGE	7
sudo reboot

