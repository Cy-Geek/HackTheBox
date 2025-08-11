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

A : **3** , SSH, FTP and HTTP

Q : After running a "Security Snapshot", the browser is redirected to a path of the format /[something]/[id], where [id] represents the id number of the scan. What is the [something]?

Browse the app on port 80, you'll find a dashboard like that, and logged in as Nathan
<img width="821" height="520" alt="Pasted image 20250217074848" src="https://github.com/user-attachments/assets/cb7fa4b4-dcfa-4af4-b906-5f5f305ea681" />

Click on the three lines up left, then click on Security snapshot

<img width="298" height="294" alt="Pasted image 20250217075005" src="https://github.com/user-attachments/assets/3414c913-20b1-409b-882d-1765b0196812" />

Take a look at the URL

<img width="414" height="42" alt="Screenshot 2025-02-17 075031" src="https://github.com/user-attachments/assets/f22a826b-7baf-445a-a039-e7ae3243754a" />

A : The word is **data**

Q : Are you able to get to other users' scans?

A : **yes**, If we change the ID in the URL we'll see other users scans

Q : What is the ID of the PCAP file that contains sensitive data?

Change the ID in the URL to 0 then download the file from here 
<img width="834" height="725" alt="Screenshot 2025-02-17 082959" src="https://github.com/user-attachments/assets/e2122256-4939-4b7d-becd-22b684bb5c5e" />

If you try to cat out the file you’ll find it encrypted, so open **Wireshark**
![](https://miro.medium.com/v2/resize:fit:700/1*VMbMW4cSzioO3Zbd_OgRow.png)

It’s gonna show you this window :
<img width="881" height="687" alt="Screenshot 2025-02-18 170005" src="https://github.com/user-attachments/assets/9355c317-64ce-4176-afbe-dda7c9c5daf0" />

Click on the **File tab up left or click ‘ctrl+o’ as a shortcut**  
<img width="354" height="432" alt="Screenshot 2025-02-18 170017" src="https://github.com/user-attachments/assets/48141cb9-08a5-438a-966c-f94ca8a09765" />

Then go to where the PCAP file was downloaded and open it
<img width="883" height="662" alt="Screenshot 2025-02-18 170031" src="https://github.com/user-attachments/assets/1e64058f-8aaf-4899-bd96-31e907f7eb14" />

You’re gonna see the following, type ftp in the bar to filter out the results
<img width="1130" height="447" alt="Screenshot 2025-02-18 170147" src="https://github.com/user-attachments/assets/0c6a3658-67f2-4fad-83bb-108bf755d7ed" />

Cool! we’ve found a login successful attempt with the user : **nathan and password** : **Buck3tH4TF0RM3!**

We can also do that in another way I learned in **CTF** by using the command `strings` to convert the file to a string
<img width="710" height="77" alt="Screenshot 2025-02-17 083533" src="https://github.com/user-attachments/assets/363ac130-d4a6-49f0-8ed8-8a0818de603d" />

Then look for any sensitive information in the output string, we’ll get the same credentials we found on **Wireshark**
<img width="312" height="174" alt="Screenshot 2025-02-17 083557" src="https://github.com/user-attachments/assets/669aae8b-8ee7-4e57-b506-a6aecf9caafa" />

A : File ID is **0**

Q : Which application layer protocol in the pcap file can the sensitive data be found in?

<img width="177" height="24" alt="Pasted image 20250217084105" src="https://github.com/user-attachments/assets/8503952e-7dd8-4786-b3c9-39f39c30e735" />

A : **FTP**, very obvious

Q : We managed to collect Nathan's FTP password. On what other service does this password work?

We found in the Nmap scan that port 22 for SSH is open so let's try to connect to it
<img width="1060" height="684" alt="Screenshot 2025-02-17 084506" src="https://github.com/user-attachments/assets/22fe144b-b75a-45e8-9ee7-8b0a8eba49f0" />
Cool! it worked!

A : `SSH`

Task : Submit the flag located in the Nathan user's home directory 

List files, we found user.txt file which contains the flag  
<img width="299" height="104" alt="Screenshot 2025-02-17 084541" src="https://github.com/user-attachments/assets/c2cb2768-da7b-4a9d-911b-746e66915541" />

We can also login with FTP using same credentials, we'll find the flag there too, download it  
<img width="803" height="461" alt="Screenshot 2025-02-17 084232" src="https://github.com/user-attachments/assets/aed05c90-2d25-4d11-a828-b08062fc6093" />

Then cat it out and here's the flag  
<img width="345" height="72" alt="Screenshot 2025-02-17 084301" src="https://github.com/user-attachments/assets/c07deb5c-c033-4f68-ae98-65a9122c7eff" />

User Flag : **1aeba54bd276f41be26d0802ba8de021**


# Privilege Escalation
**The most fun part**

Q : What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

We're gonna use LinPEAS for that, so we need to transfer the file from our shell to Nathan's shell

Establish an HTTP python server on port 80 in your machine  
<img width="520" height="84" alt="Pasted image 20250217091523" src="https://github.com/user-attachments/assets/12253972-d71e-4f03-9602-1c39ce56f20a" />

In Nathan shell, grab **linpeas.sh** file from our machine use `wget` command, and make it executable  
 <img width="771" height="438" alt="Pasted image 20250217091658" src="https://github.com/user-attachments/assets/a5615f8c-864a-4784-a367-2e5e8d337752" />

Then go ahead and run it  
<img width="794" height="706" alt="Pasted image 20250217091809" src="https://github.com/user-attachments/assets/40dc76ed-aa45-4647-9ca6-a52c136d94b4" />

Look for anything interesting in the output, and don't forget this is the color rank for the findings
<img width="729" height="142" alt="Pasted image 20250217092032" src="https://github.com/user-attachments/assets/4d2f9eb3-a87f-483b-ac8e-9fb22200bb6d" />

We found this **red/yellow** file!  
<img width="997" height="128" alt="Screenshot 2025-02-17 091336" src="https://github.com/user-attachments/assets/fd8c6415-6518-412d-bcfe-bc441b9121b1" />

Get the capabilities of that file to ensure it has special ones  
<img width="434" height="36" alt="Screenshot 2025-02-17 110747" src="https://github.com/user-attachments/assets/d29b6df0-7b49-4e09-92a8-75c312fcdf3d" />

A : The full path is  **/usr/bin/python3.8**

The real question is 'What do **cap_setuid,cap_net_bind_service+eip** mean'?  
<img width="664" height="221" alt="Screenshot 2025-02-17 094925" src="https://github.com/user-attachments/assets/35113842-5024-4030-b999-20ef4a32966c" />

For more info do this command `man capabilities` OR Visit **[man7](https://man7.org/linux/man-pages/man7/capabilities.7.html)**

Task : Submit the flag located in root's home directory

##### I didn't wanna just copy and paste from **[GTFOBins](https://gtfobins.github.io/)** so I went through a lot of search to understand, But you can do whatever you prefer, the choice is yours

##### Pay attention to the following

The id of the **root** user is **0** by default in linux systems, since we wanna get a root shell we need to set our id to 0
And since the **python3.8** binary file supports **CAP_SETUID** as we saw, we can run a command in python3 that set our id to 0 then a another command to establish a shell.

`/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'`

We'll separate this command to couple of parts.
1. `/usr/bin/python3.8` : Full path to our file which you can actually just type **python3.8** or **python3**
2. `-c `: A short for command to make you run commands 
3. `import os `: Import a module or a library to run command on the system
4. `os.setuid(0)` : That function **setuid()** sets our user id to 0
5. `os.system("/bin/bash")` : This function establish a bash shell which is gonna be root shell 
<img width="772" height="115" alt="Screenshot 2025-02-17 113638" src="https://github.com/user-attachments/assets/876cd042-cc95-4936-aa3c-3c6a6ca6a60b" />


Same thing in the python IDLE, the `-c` flag saves us all that  
<img width="640" height="168" alt="Pasted image 20250217115240" src="https://github.com/user-attachments/assets/6e26ff30-e178-4d8c-8486-8ea131021096" />

Then cat out the root flag  
<img width="309" height="38" alt="Pasted image 20250217115524" src="https://github.com/user-attachments/assets/9adb5ea7-322f-474f-b85e-10cf4727e73a" />

Root Flag : **9e02256fad08d2eda01fe02952e929cb**

**Congrats! You pawned the machine successfully!**

<img width="702" height="644" alt="Pasted image 20250729102237" src="https://github.com/user-attachments/assets/bed25d81-a173-4c97-93ea-b9f64c9d10f8" />

