# Blue Team

We will be investigating the incident using Kibana to analyze the logs that took place when Red team attacked. We will be viewing logs on Kibana by importing filebeat, metricbeat and packetbeat data, but first we will have to add Kibana log datas and then create a dashboard for visualization.

<details>
<summary> 1. Identify the offensive traffic </b> </summary>

Identify the traffic between your machine and the web machine:

- Run the command on the Discover page of Kibana: `source.ip: 192.168.1.90 and destination.ip: 192.168.1.105` which indicates the source IP of Kali machine and your destination machine (your web server).
- Run `url.path: /company_folders/secret_folder/`

![image](https://github.com/user-attachments/assets/989e6634-e1f6-4358-9bbb-b68bb8f9bdf8)

- The following responses: `401, 301, 200, 207, 303` returned shown in the images below:

![image](https://github.com/user-attachments/assets/0dff360c-8584-4454-acf2-8c69145befa6)

Identifying the Port scan:

- The port scan (192.168.1.90) occurred on July 23, 2022 @ 15:00.
- There were a total of 133,288 packets sent from 192.168.1.90.
- There was an increased activity spike in the network traffic that helps identify the port scans.
- We can see a spike in the Connections over time [Packetbeat Flows] ECS and Errors vs successful transactions [Packetbeat] ECS.

</details>	





<details>
<summary> 2. Find the Request for the Hidden Directory </b> </summary> 
Looking at the interaction between the attacking machine with the webserver.

- Request occurred on July 23, 2022 @ ~15:00. The secret_folder was requested 16,213 times, as shown in the Top 10 HTTP requests [Packetbeat] ECS panel.
- Files within the secret_folder was obtained when logging into Ashton's account which then lead us to connect_to_corp_server and contained sensitive information.
- Inside the secret folder revealed sensitive information on Ryan’s account password and instructions on how to navigate into Ryan’s webDAV server.

![image](https://github.com/user-attachments/assets/504f6491-61cf-46ac-a206-40c880a90b00)

![image](https://github.com/user-attachments/assets/5d41bed7-010f-4f61-8432-875e27bdf02b)

![image](https://github.com/user-attachments/assets/e43044b2-9d5a-4770-ac58-7a5fedcfe4c4)

Mitigation:
<blockquote>
  <strong>What kind of alarm would you set to detect this behavior in the future?</strong><br>
</blockquote>

- Set an alarm alert that goes off for any machine that attempts to access the directory or file.
- Set an alarm that sets off when a user from non-whitelisted IP address tries to access directory.
- Setting a threshold of 2-3 attempts every 20 minutes that would trigger an alert to be sent to SOC analyst.

<blockquote>
  <strong>Identify at least one way to harden the vulnerable machine that would mitigate this attack.</strong><br>
</blockquote>

- Directory file should be removed from the server.
- Store files in the central database and not directly in web server file systems and definite own resource names used to access the files.
- Whitelisting permitted name and/or characters of file names or paths from user inputs. Blacklisting characters to filter out ../ and strings not recommended.
- Mitigating vulnerability on web server side, ensure using up-to-date web server software. Running minimum privileges and only have access to directories that the website or application actually needs.
- Detecting these vulnerabilities by regularly scan your websites and web applications.
- Encrypt data file that are confidential.  
</details>	





<details>
<summary> 3. Identify the Brute Force Attack </b> </summary>
After identifying the hidden directory, Hydra was used to brute-force the target server.

Packets from Hydra was identified using the following search functions on the Discovery page of Kibana:

- search: `url.path: /company_folders/secret_folder/` and look through results and notice `Hydra` is identified under `user_agent.original` as shown in the image below:

![image](https://github.com/user-attachments/assets/edcbf1a1-86d0-4c93-bca9-64322020588a)

- search: `source.ip: 192.168.1.90 AND destination.ip:192.168.1.105 AND http.response.status_code:401 AND url.path:/company_folders/secret_folder AND user_agent.original:"Mozilla/4.0 (Hydra)"`

![image](https://github.com/user-attachments/assets/2ec4b961-0044-4384-99fe-ea6722395b06)

- There were 16,205 requests made in the attack. Within the 16,205 requests, 2 requests was made before discovering the password as shown illustrated in HTTP Transactions [PacketBeat] ECS panel.
- The HTTP status codes for the top queries [PacketBeat] ECS panel shows the breakdown of 401 unauthorized status codes as opposed to 200 OK status codes.

![image](https://github.com/user-attachments/assets/6dad7eca-5954-425b-903d-79fd35009656)

- The Connections over time [Packetbeat Flows] ECS panel shows a connection spike.

![image](https://github.com/user-attachments/assets/97686003-c43e-4509-993f-58c27ab08d68)

Mitigation:

<blockquote>
  <strong>What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?</strong><br>
</blockquote>

- Set an alert if 401 unauthorized status code is returned back from any server.
- Set threshold of 10 login attempts per hour and refine from there.
- Set alert if user_agent.original value includes Hydra in the name.

<blockquote>
  <strong>Identify at least one way to harden the vulnerable machine that would mitigate this attack.</strong><br>
</blockquote>

- Create a password policy for the company - an assigned unique user account and password requirements such as new passwords to be created and will expire every 90 days and must be changed.
- Accounts shall be locked after six failed login attempts within 30 minutes and shall remain locked for at least 30 minutes or until the System Administrator unlocks the account.
- Apply the NIST 800-63B framework for password requirements. Limit failed login attempts and logins to specific IP address or range.
- Strong protected passwords using Captcha and Two-Factor Authentication.  
</details>	




<details>
<summary> 4. Find the WebDav Connection </b> </summary>
- In the Top 10 HTTP requests [Packetbeat] ECS panel, 98 requests were made in the webDAV directory and 52 requests were made in the webDAV/shell.php.
  
- Within the webDAV directory, two files found named passwd.dav and shell.php.

![image](https://github.com/user-attachments/assets/9b32c65b-3da2-457e-ade7-d9095ffc418c)

Mitigation:

<blockquote>
  <strong>What kind of alarm would you set to detect such access in the future?</strong><br>
</blockquote>

- Set an alert each time another machine other than main machine accessing the directory.
- Set a threshold of > 0 whenever resources from webDAV is accessed from an external IP address

<blockquote>
  <strong>Identify at least one way to harden the vulnerable machine that would mitigate this attack.</strong><br>
</blockquote>

- WebDAV operates over the web via HTTP, securing transactions with SSL to switch the site to HTTPS schema. The webserver will be able to negotiate connections with HTTPS instead of HTTP.
- Using a vulnerability management tool such as Automated Vulnerability Detection System (AVDS) to detect webDAV in your web application.
- Disabling webDAV when not in use.
- Web application firewall with a rule that restrict access to shared folder.
- Connections to this shared folder should not be accessible from the web interface.
  
</details>	





<details>
<summary> 5. Identify the Reverse Shell and meterpreter Traffic </b> </summary>
A PHP reverse shell to the targets machine and started a meterpreter shell session.

To identify the meterpreter session, on the Discovery page of Kibana, we can use the search function:

- `source.ip: 192.168.1.90 AND destination.ip:192.168.1.105 AND query:"GET /webdav/shell.php"`
- `source.ip: 192.168.1.105 and destination.port: 4444`

![image](https://github.com/user-attachments/assets/448f320a-14fa-4cf8-8e8d-22abc8091bc0)

Mitigation:

<blockquote>
  <strong>What kinds of alarms would you set to detect this behavior in the future?</strong><br>
</blockquote>

- Set an alert for any traffic moving over port 4444.
- Set an alert threshold of one attempt for any .php file that is uploaded to a server.

<blockquote>
  <strong>Identify at least one way to harden the vulnerable machine that would mitigate this attack.</strong><br>
</blockquote>

- Removing the ability to upload files to this directory over the web interface would take care of this issue. Store uploaded files in a location not accessible from the web
- Only allow users with authentication to upload files and define valid types of files that the users should be allowed to upload.
- Improve web application security with web application firewalls
- Company should implement NISTIR 7316 framework for assess control management. 

</details>	
