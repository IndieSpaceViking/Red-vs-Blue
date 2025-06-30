# Monitoring Setup

I will be installing FileBeat, MetricBeat and PacketBeat onto our Capstone VM, so we can collect the logs as the attack is taking place in our server.

## Configuration

### FileBeat

Setting up filebeat ships log data (simplifies, parsing, visualization of log formats) from server to ELK Stack or monitoring VM
Commands:
- `filebeat modules enable apache`
- `filebeat setup`
  
Output:

![image](https://github.com/user-attachments/assets/8038d053-4e30-4887-941d-ed166dc2fc3b)

### MetreicBeat

Setting up metricbeat ships metrics data from server (e-g MongoDB, MYSQL, Apache) to ELK Stack or monitoring VM
Commands:
- `metricbeat modules enable apache`
- `metricbeat setup`
  
Output:

![image](https://github.com/user-attachments/assets/26d6464d-3813-429f-8c58-1827605486e2)

### PacketBeat

Setting up packetbeat integrates Elasticsearch and Kibana to provide realtime analysis
- `packetbeat setup`

Output:

![image](https://github.com/user-attachments/assets/6d1d6668-05f5-485e-a959-44622d4b8a98)


Restart all 3 services. Run the following commands:
- `systemctl restart filebeat`
- `systemctl restart metricbeat`
- `systemctl restart packetbeat`

Note:These restart commands should not give any output:

![image](https://github.com/user-attachments/assets/b0def3f9-d186-4e64-b6f9-1ec3fc323f75)

Once all three of these have been enabled, I will close the terminal window for this machine and proceed with my attack.
