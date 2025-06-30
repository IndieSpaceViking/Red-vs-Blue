# Blue Team

We will be investigating the incident using Kibana to analyze the logs that took place when Red team attacked. We will be viewing logs on Kibana by importing filebeat, metricbeat and packetbeat data, but first we will have to add Kibana log datas and then create a dashboard for visualization.

<details>
<summary> Click here to view on how to add Kibana Log Data and Creating a Dashboard for Visualization </b> </summary>

### Adding Kibana Log Data
To start viewing logs in Kibana, we will need to import our filebeat, metricbeat and packetbeat data.

Double-click the Google Chrome icon on the Windows host's desktop to launch Kibana. If it doesn't load as the default page, navigate to http://192.168.1.105:5601.

This will open 4 tabs automatically, but for now, we only want to use the first tab.

Click on the `Explore My Own` link to get started.

### Adding Appache logs
- Click on `Add Log Data`
- Click on `Apache logs`
- Scroll to the bottom of the page.
- Click on `Check Data` You should see a message highlighted in green: `Data successfully received from this module`

![image](https://github.com/user-attachments/assets/9ecff899-23ba-444c-9d96-8ab002fbace1)


Return to the Home screen by moving back 2 pages.

### Adding System Logs
- Click on `Add Log Data`
- Click on `System logs`
- Scroll to the bottom of the page.
- Click on `Check Data` You should see a message highlighted in green: `Data successfully received from this module`

![image](https://github.com/user-attachments/assets/cee99ab3-4dc8-436d-8efb-eca06299b3c8)

Return to the Home screen by moving back 2 pages.

### Adding Apache Metrics
- Click on `Add Metric Data`
- Click on `Apache Metrics`
- Scroll to the bottom of the page.
- Click on `Check Data` You should see a message highlighted in green: `Data successfully received from this module`

![image](https://github.com/user-attachments/assets/e284ed49-8d26-4830-9b00-5bdd63c2edf1)

Return to the Home screen by moving back 2 pages.

### Adding System Metrics
- Click on `Add Metric Data`
- Click on `System Metrics`
- Scroll to the bottom of the page.
- Click on `Check Data` You should see a message highlighted in green: `Data successfully received from this module`

![image](https://github.com/user-attachments/assets/57768440-f2bb-4348-87ef-934898107c94)

Close Google Chrome and all of it's tabs. Double click on Chrome to re-open it.

### Dashboard Creation
We will create visualization of our data to write for report.

- Click on Dashboards on the left navigation panel.
- Click on Create Dashboard in right upper hand side.
On the new page click on Add an existing to add the following existing reports:

- `HTTP status codes for the top queries [Packetbeat] ECS`
- `Top 10 HTTP requests [Packetbeat] ECS`
- `Network Traffic Between Hosts [Packetbeat Flows] ECS`
- `Top Hosts Creating Traffic [Packetbeat Flows] ECS`
- `Connections over time [Packetbeat Flows] ECS`
- `HTTP error codes [Packetbeat] ECS`
- `Errors vs successful transactions [Packetbeat] ECS`
- `HTTP Transactions [Packetbeat] ECS`
After adding the dashboard it should look like below images:

![image](https://github.com/user-attachments/assets/6e003aef-4fd6-4660-867d-68d3acd775b1)

![image](https://github.com/user-attachments/assets/b61e26a3-a830-4a2f-b84a-9dbd9eebb609)
</details>	





## Log Analysis
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



