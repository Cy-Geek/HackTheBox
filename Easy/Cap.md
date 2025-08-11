<img width="1266" height="144" alt="Pasted image 20250811155040" src="https://github.com/user-attachments/assets/9f268d65-1812-486e-a8d8-6f0e197e4348" />


### Stuff to learn in this box 
	1. IDOR vulnerability
	2. Intro to Wireshark Sniffer
	3. Some useful Linux commands
	4. FTP & SSH Protocols
	5. Privilege Escalation with LinPEAS
	6. Capabilities in Linux

Q : How many TCP ports are open?

Common lazy use Nmap, You can figure that out by yourself I believe in you
![[200 1 1.webp]]
A : **3** , SSH, FTP and HTTP

Q : After running a "Security Snapshot", the browser is redirected to a path of the format /[something]/[id], where [id] represents the id number of the scan. What is the [something]?

Open the app on port 80, you'll find a dashboard like that, and logged in as Nathan![[Pasted image 20250217074848.png]]
Click on the three lines up left, then click on Security snapshot
![[Pasted image 20250217075005.png]]
Take a look at the URL
![[Screenshot 2025-02-17 075031.png]]
A : The word is **data**

Q : Are you able to get to other users' scans?
A : **yes**, If we change the ID in the URL we'll see other users scans

Q : What is the ID of the PCAP file that contains sensitive data?

Change the ID in the URL to 0 the download the file from here ![[Screenshot 2025-02-17 082959.png]]
If you try to cat out the file you’ll find it encrypted, so open **Wireshark**
![](https://miro.medium.com/v2/resize:fit:700/1*VMbMW4cSzioO3Zbd_OgRow.png)

It’s gonna show you this window :
![[Screenshot 2025-02-18 170005.png]]

Click on the **File tab up left or click ‘ctrl+o’ as a shortcut**
![[Screenshot 2025-02-18 170017.png]]

Then go to where the PCAP file was downloaded and open it
![[Screenshot 2025-02-18 170031.png]]

You’re gonna see the following, type ftp in the bar to filter out the results
![[Screenshot 2025-02-18 170147.png]]

Cool! we’ve found a login successful attempt with the user : **nathan and password** : **Buck3tH4TF0RM3!**

We can also do that in another way I learned in **CTF** by using the command `strings` to convert the file to a string![[Screenshot 2025-02-17 083533.png]]

Then look for any sensitive information in the output string, we’ll get the same credentials we found on **Wireshark**
![[Screenshot 2025-02-17 083557.png]]
![[giphy 1 1.webp]]
A : File ID is **0**

Q : Which application layer protocol in the pcap file can the sensitive data be found in?

![[Pasted image 20250217084105.png]]
A : **FTP**, very obvious from the structure

Q : We managed to collect Nathan's FTP password. On what other service does this password work?

We found in the Nmap scan that port 22 for SSH is open so let's try to connect to it
![[Screenshot 2025-02-17 084506.png]]
Cool! it worked!
![[200 2.webp]]
A : `SSH`

Task : Submit the flag located in the Nathan user's home directory 

List files, we found user.txt file which contains the flag
![[Screenshot 2025-02-17 084541.png]]

We can also login with FTP using same credentials, we'll find the flag there too, download it
![[Screenshot 2025-02-17 084232.png]]
Then cat it out and here's the flag
![[Screenshot 2025-02-17 084301.png]]
User Flag : **1aeba54bd276f41be26d0802ba8de021**

![[200 6.webp]]
#### Privilege Escalation
**The most fun part**
![[200 4.webp]]

Q : What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

We're gonna use LinPEAS for that, so we need to transfer the file from our shell to Nathan's shell

Establish an HTTP python server on port 80 in your machine![[Pasted image 20250217091523.png]]

In Nathan shell, grab **linpeas.sh** file from our machine use `wget` command, and make it executable![[Pasted image 20250217091658.png]]

Then go ahead and run it ![[Pasted image 20250217091809.png]]
Look for anything interesting in the output, and don't forget this is the color rank for the findings![[Pasted image 20250217092032.png]]

We found this **red/yellow** file!![[Screenshot 2025-02-17 091336.png]]
![[200 5.webp]]

Get the capabilities of that file to ensure it has it ![[Screenshot 2025-02-17 110747.png]]
A : The full path is  **/usr/bin/python3.8**

The real question is 'What do **cap_setuid,cap_net_bind_service+eip** mean'?
![[Screenshot 2025-02-17 094925.png]]
For more info do this command `man capabilities` OR Visit **[man7](https://man7.org/linux/man-pages/man7/capabilities.7.html)**

Task : Submit the flag located in root's home directory

##### I didn't wanna just copy and paste from **[GTFOBins](https://gtfobins.github.io/)** so I went through a lot of search to understand, But you can do whatever you prefer, the choice is yours

##### Pay attention to the following
![[200 1.webp]]
The id of the **root** user is **0** by default, since we wanna get a root shell we need to set our id to 0
And since the **python3.8** binary file supports **CAP_SETUID** as we saw, we can run a command in python3 that set our id to 0 then a another command to establish a shell.

`/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'`
We'll separate this command to couple of parts.
`/usr/bin/python3.8` : Full path to our file which you can actually just type **python3.8** or **python3**
`-c `: A short for command to make you run commands 
`import os `: Import a module or a library to run command on the system
`os.setuid(0)` : That function **setuid()** sets our user id to 0
`os.system("/bin/bash")` : This function establish a bash shell which is gonna be root shell 
![[Screenshot 2025-02-17 113638.png]]
Same thing in the python IDLE, the `-c` flag saves us all that ![[Pasted image 20250217115240.png]]

Then cat out the root flag
![[Pasted image 20250217115524.png]]
Root Flag : **9e02256fad08d2eda01fe02952e929cb**

**Congrats! You pawned the machine successfully!**
![[giphy 2.webp]]


![[Pasted image 20250729102237.png]]
