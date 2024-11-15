# cyberpatriot-checklist
yea no this is something else

best opening checklist rn
# Sean "Forty-Bot" Anderson's 0x539 Linux Checklist v1.0

## Notes

**If a command errors or fails, try it again with `sudo` (or `sudo !!` to save typing)**

**Google anything and everything. If you don't know or understand something, google it**

When you see the syntax `$word`, do not type it verbatim, but instead substitute the appropriate word (usually referenced in a previous command).

When the order of steps does not matter, bullet points have been used instead of ordinals.

To edit files, run `gedit`, a graphical editor akin to notepad; `nano`, a simple command-line editor; or `vim`, a powerful  but less intuitive command-line editor. Note that vim may need to be installed with `apt-get install vim`.

## Checklist

1. Read the readme
	
	Note down which ports/users are allowed.

1. **Do Forensics Questions**
	
	You may destroy the requisite information if you work on the checklist!

1. Secure root

	set `PermitRootLogin no` in `/etc/ssh/sshd_config`

1. Secure Users
	1. Disable the guest user.
	
		Go to `/etc/lightdm/lightdm.conf` and add the line
		
		`allow-guest=false`

		Then restart your session with `sudo restart lightdm`. This will log you out, so make sure you are not executing anything important.

	1. Open up `/etc/passwd` and check which users
		* Are uid 0
		* Can login
		* Are allowed in the readme
	1. Delete unauthorized users:
		
		`sudo userdel -r $user`

		`sudo groupdel $user`
	1. Check `/etc/sudoers.d` and make sure only members of group sudo can sudo.
	1. Check `/etc/group` and remove non-admins from sudo and admin groups.
	1. Check user directories.
		1. cd `/home`
		1. `sudo ls -Ra *`
		1. Look in any directories which show up for media files/tools and/or "hacking tools."
	1. Enforce  Password Requirements.
		1. Add or change password expiration requirements to `/etc/login.defs`.
			
			```
			PASS_MIN_DAYS 7
			PASS_MAX_DAYS 90
			PASS_WARN_AGE 14
			```
		1. Add a minimum password length, password history, and add complexity requirements.
			1. Open `/etc/pam.d/common-password` with sudo.
			1. Add `minlen=8` to the end of the line that has `pam_unix.so` in it.
			1. Add `remember=5` to the end of the line that has `pam_unix.so` in it.
			1. Locate the line that has pam.cracklib.so in it. If you cannot find that line, install cracklib with `sudo apt-get install libpam-cracklib`.
			1. Add `ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-` to the end of that line.
		3. Implement an account lockout policy.
			1. Open `/etc/pam.d/common-auth`.
			2. Add `deny=5 unlock_time=1800` to the end of the line with `pam_tally2.so` in it.
		4. Change all passwords to satisfy these requirements.
			
			`chpasswd` is very useful for this purpose.

1. Enable automatic updates
	
	In the GUI set Update Manager->Settings->Updates->Check for updates:->Daily.

1. Secure ports
	1. `sudo ss -ln`
	1. If a port has `127.0.0.1:$port` in its line, that means it's connected to loopback and isn't exposed. Otherwise, there should only be ports which are specified in the readme open (but there probably will be tons more).
	1. For each open port which should be closed:
		1. `sudo lsof -i :$port`
		1. Copy the program which is listening on the port.
		`whereis $program`
		1. Copy where the program is (if there is more than one location, just copy the first one).
		`dpkg -S $location`
		1. This shows which package provides the file (If there is no package, that means you can probably delete it with `rm $location; killall -9 $program`).
		`sudo apt-get purge $package`
		1. Check to make sure you aren't accidentally removing critical packages before hitting "y".
		1. `sudo ss -l` to make sure the port actually closed.

1. Secure network
	1. Enable the firewall
	
		`sudo ufw enable`
	1. Enable syn cookie protection
	
		`sysctl -n net.ipv4.tcp_syncookies`
	1. Disable IPv6 (Potentially harmful)
	
		`echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf`
	1. Disable IP Forwarding
		
		`echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward`
	1. Prevent IP Spoofing
	
		`echo "nospoof on" | sudo tee -a /etc/host.conf`

1. Install Updates

	Start this before half-way.

	* Do general updates.
		1. `sudo apt-get update`.
		1. `sudo apt-get upgrade`.

	* Update services specified in readme.
		1. Google to find what the latest stable version is.
		1. Google "ubuntu install service version".
		1. Follow the instructions.
	
	* Ensure that you have points for upgrading the kernel, each service specified in the readme, and bash if it is [vulnerable to shellshock](https://en.wikipedia.org/wiki/Shellshock_%28software_bug%29).
	

1. Configure services
	1. Check service configuration files for required services.
		Usually a wrong setting in a config file for sql, apache, etc. will be a point.
	1. Ensure all services are legitimate.
		
		`service --status-all`

1. Check the installed packages for "hacking tools," such as password crackers.

1. Run other (more comprehensive) checklists. This is checklist designed to get most of the common points, but it may not catch everything.
 
## Tips

* Netcat is installed by default in ubuntu. You will most likely not get points for removing this version.
* Some services (such as `ssh`) may be required even if they are not mentioned in the readme. Others may be points even if they are explicitly mentioned in the readme

## Extra Stuff I Found In Training Round 2

1. Account Lockout Policy
   1. `sudo touch /usr/share/pam-configs/faillock`
   2. `sudo gedit /usr/share/pam-configs/faillock`
      ** Note: the file WILL be empty
   3. Type this whole text into the file
      ```
      Name: Enforce failed login attempt counter
      Default: no
      Priority: 0
      Auth-Type: Primary
      Auth:
      [default=die] pam_faillock.so authfail
      sufficient pam_faillock.so authsucc
      ```

   1. `sudo touch /usr/share/pam-configs/faillock_notify`
   2. `sudo gedit /usr/share/pam-configs/faillock_notify`
   3. Type this whole text into the file
      ```
      Name: Notify on failed login attempts
      Default: no
      Priority: 1024
      Auth-Type: Primary
      Auth:
      requisit pam_faillock.so preauth
      ```

   1. `sudo pam-auth-update`
   2. Select the Notify on failed login attempts and Enforce failed login attempt counter to enable.
   3. Select ok
  
2. Check for null password authentication
   1. `sudo gedit /etc/pam.d/common-auth`
   2. Look for `auth [success=2 default=ignore] pam_unix.so nullok` and remove the nullok option.

(incase the other guide did not work for some reason)
3. Enable IPv4 TCP SYN cookies & Disable IPv4 Port Forwarding
   1. `sudo gedit /etc/sysctl.conf`
   2. Change `net.ipv4.tcp_syncookies=0` to `net.ipv4.tcp_syncookies=1`
   3. Change `net.ipv4.ip_forward=0` to `net.ipv4.ip_forward=1`
   4. `sudo sysctl --system` to apply these settings

4. Check for insecure permissions on shadow files (and maybe other files too)
   1. `ls -alF /etc/shadow` (or more generally `ls -alF $file`) to view permissions.
      ** Check (https://linuxhandbook.com/linux-file-permissions/) for more information on what permissions are
      ** More specifically, if there is anything on the last 3 characters of the permissions on sensitive files, then you know something is wrong
   2. `sudo chmod 640 /etc/shadow` (or more generally `ls chmod $permissions $file`) to fix permissions
      ** 640 refers to (-rw-r----) or just saying no one else can read the file

5. Disable insecure or unneccessary services.
   1. Honestly tbh, this is just like you know it or not. But maybe trial and error can just brute force the answer lol 💀
      Here are some services to look out for:
      
      	BAD STUFF
      
      	`john, nmap, vuze, frostwire, kismet, freeciv, minetest, minetest-server, medusa, hydra, truecrack, ophcrack, nikto, cryptcat, nc, netcat, tightvncserver, x11vnc, nfs, xinetd`
	
 		POSSIBLY BAD STUFF

   		`samba, postgresql, sftpd, vsftpd, apache, apache2, ftp, mysql, php, snmp, pop3, icmp, sendmail, dovecot, bind9, nginx`

   		MEGA BAD STUFF

		`telnet, rlogind, rshd, rcmd, rexecd, rbootd, rquotad, rstatd, rusersd, rwalld, rexd, fingerd, tftpd, telnet, snmp, netcat, nc`

   2. If you want to disable one of these services just type `sudo systemctl disable --now $service`

7. Removing netcat backdoor (Very Specific)
   1. `sudo ss -tlnp`
   2. Look for suspicious ports that are being listened on (e.g. ...:1337)
   3. `sudo gedit /etc/crontab` ** Honestly you should just be checking this anyway to look for suscpious cron jobs that are throughout the system
   4. Remove nc.traditional line
   5. `sudo pkill -f nc.traditional`
   6. `which nc.traditional`
   7. `sudo rm /usr/bin/nc.traditional`
