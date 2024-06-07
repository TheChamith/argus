# SIEM-on-ELK-Stack
We are using Suricata and ELK stack to make this SIEM.
There are two seperate VMs, one dedicated for the Suricata and the other is to run the ELK Stack.
Additionally a Kali Machine is used to attack the Suricata machine to generate security logs and for testing purposes.

Follow this link to more information.
https://mysliit-my.sharepoint.com/:w:/g/personal/it21308802_my_sliit_lk/EaHz6ApDg-5AqsUe6jZte_8BaLk7mAs-_iD6TUuuYuL1mQ?e=Igv68T

<h2>Prerequires</h2>
<ul>
  <li>The only prerequisite for this lab is knowing how to create a virtual machine with an ISO image.</li>
<li>This guide will not cover the process of creating a virtual machine.</li>
<li>Different virtualization software have unique user interfaces and options for creating virtual machines.</li>
<li>There are many tutorials and guides available that provide step-by-step instructions for creating a VM with an ISO image.</li>
<li>If you're having a hard time finding a guide, follow this tutorial on GeeksforGeeks.</li>
https://www.geeksforgeeks.org/how-to-install-ubuntu-on-virtualbox/
</ul>


<h3>Installing Suricata</h3>

To get started, we will add the Open Information Security Foundation’s (OSIF) repository to our Ubuntu machine:
``` bash
$ sudo add-apt-repository ppa:oisf/suricata-stable
```
![1](https://github.com/TheChamith/argus/assets/99456721/0e264fd7-9985-404b-a6f5-9fcfde6a7c66)

Next, update the repository we just added:
```bash
$ sudo apt-get update
```
![2](https://github.com/TheChamith/argus/assets/99456721/008d281c-756d-457a-a819-8857822882d0)

Now, install Suricata using the following command:

```bash
$ sudo apt-get install suricata
```
![image](https://github.com/TheChamith/argus/assets/99456721/6c970452-5afa-433e-9774-a7d048a13ec4)
Just like that, you now have Suricata installed on your Ubuntu machine. Now, this is where the fun stuff comes into play. The next steps will show how to configure Suricata.

First, enable the suricata.service to start at boot. This will allow Suricata to run whenever the machine restarts:

```bash
$ sudo systemctl enable suricata.service
```
![image](https://github.com/TheChamith/argus/assets/99456721/d6bd8012-dbea-4ccd-8fff-88b0e3958536)
Run the following command to verify the Suricata service is running:

```bash
$ sudo systemctl status suricata.service
``` 

![image](https://github.com/TheChamith/argus/assets/99456721/e57041b5-0381-4cb7-859d-05b4f6c88ac8)

As you can see, Suricata is currently running in IDS (af-packet) mode

The ifconfig command will show all the available interfaces
To configure the suricata.yaml file, note that Suricata inspects traffic on eth0 by default. If your system uses a different interface, update this value accordingly.
```bash
$  ifconfig 
```
![image](https://github.com/TheChamith/argus/assets/99456721/ab45a291-19d3-486f-a829-184800d2ea58)


Another alternative command is to show your default network interface is the following:
```bash
ip -p -j route show default 
``` 
![image](https://github.com/TheChamith/argus/assets/99456721/58544d3b-c414-4aa6-98d2-858462e4d0e3)


Edit the suricata.yaml configuration file.
```bash
$ sudo nano /etc/suricata/suricata.yaml
```
![image](https://github.com/TheChamith/argus/assets/99456721/2428abd3-e65c-4bbb-9e00-885c67f20b06)

Configuring the correct interface in the config file (ens33)
![image](https://github.com/TheChamith/argus/assets/99456721/4919e0a3-1f0a-4233-9740-a5a0ce74fcb2)


Adding rules to config file
![image](https://github.com/TheChamith/argus/assets/99456721/f62f993e-f197-43fb-b3e0-2a9c40c468f2)


Killing the process and restaring
```bash
$ sudo kill -usr2 $(pidof suricata)
$ sudo service suricata restart
```
![image](https://github.com/TheChamith/argus/assets/99456721/ec07383a-e891-4d20-8fdb-049eca218369)

Updating suricata
```bash
$ sudo suricata-update
```
![image](https://github.com/TheChamith/argus/assets/99456721/0aabada2-2c5a-4b2a-a647-d8a0b6fbc5a2)


Updating sources
```bash
$ sudo suricata-update --no-check-certificate update-sources 
```
![image](https://github.com/TheChamith/argus/assets/99456721/b594b84f-bf76-45ad-ac1d-27601c3a28c6)


Listing the rule providers
```bash
$ sudo suricata-update list-sources
```
![image](https://github.com/TheChamith/argus/assets/99456721/a6c3ed5b-1812-48cb-b098-451254f349b6)


fetch the et/open rule
```bash
$ sudo suricata-update enable-source et/open
```
![image](https://github.com/TheChamith/argus/assets/99456721/90d5ed00-1a0d-47df-93c8-a2c95dae7f6d)


Run the suricata-update command again to load et/open ruleset and then run the kill command to update the rules without restarting.
```bash
$ sudo suricata-update 
$ sudo kill -usr2 $(pidof suricata)
```

Test Suricata has zero errors once more. 
```bash
$ sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
![image](https://github.com/TheChamith/argus/assets/99456721/6ec742fe-2609-46a8-a527-9b381ebf4f80)


Testing with a curl
```bash
$ curl http://testmynids.org/uid/index.html
```
![image](https://github.com/TheChamith/argus/assets/99456721/a246054d-134c-4b43-9767-36b6022aa35f)


Tailing and looking at the log.

```bash
$ tail /var/log/suricata/fast.log 
```
![image](https://github.com/TheChamith/argus/assets/99456721/9de79945-fd36-4bd4-a2dc-19bbbe07afe7)


Viewing the log in JSON format

```bash
$ sudo apt install jq
$ jq 'select(.alert .signature=="GPL ATTACK_RESPONSE id check returned root")' /var/log/suricata/eve.json
```
![image](https://github.com/TheChamith/argus/assets/99456721/a81f5f48-c4b4-47ba-b6a5-17aecdadecdb)


<h3>Configuring Suricata to IPS</h3>

Opening the config file to make changes

```bash
$ sudo nano /etc/default/suricata
```
![image](https://github.com/TheChamith/argus/assets/99456721/370eecd8-04ac-4666-90e5-bac1279ca6d2)


Find the LISTENMODE=af-packet and comment it out with a #. Then add the following line:
LISTENMODE=nfqueue 

![image](https://github.com/TheChamith/argus/assets/99456721/0898e3d2-87c9-4022-9ded-23ec2f0e5d63)


Restart Suricata and check the status of the Suricata service. 

```bash
$ sudo service suricata restart
$ sudo service suricata status
```

![image](https://github.com/TheChamith/argus/assets/99456721/4287e807-2e21-4cb3-9132-f295f0d1686e)

We will see that Suricata is running in IPS mode now

Open IPv4 rules with the following command:

```bash
$ sudo nano /etc/ufw/before.rules
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Add the following lines:

-I INPUT -j NFQUEUE <br>
-I OUTPUT -j NFQUEUE

Enable the firewall to load the new rules we just configured. 

```bash
$ sudo ufw enable 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Creating local rules to drop traffic.


Create the following rule – the rule we made will drop any ICMP request to our host.<br>
drop ICMP any any -> $HOME_NET any (msg: “ICMP Request Blocked”; sid:2; rev:1;) 
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
add the newly created rules to Suricata’s config file. 

```bash
$ sudo nano /etc/suricata/suricata.yaml   
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Reload Suricata’s rules without restarting with the following command:

```bash
$ sudo kill -usr2 $(pidof suricata) 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Test your configuration 

```bash
$ sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>


Pinging from the Kali machine
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
The packets are getting dropped as shown above

Hop back over to your Suricata machine and type the following command to see your logs:

```bash
$ tail /var/log/suricata/fast.log
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

We can tail it and see that it says ICMP were dropped.

<h2>Integrating Suricata with ELK Stack</h2>

Install the Elasticsearch Debian package:

```bash
$ sudo apt-get update && sudo apt-get install elasticsearch
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Configuring the elastic config file

```bash
$ sudo nano /etc/elasticsearch/elasticsearch.yml
```

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Disabling these features to send http traffic
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Enable the Elasticsearch service.

```bash
$ sudo systemctl start elasticsearch.service
```

Send a curl request to our Elasticsearch Host with the following command:

```bash
$ curl -X GET “192.168.1.10.14”
```

Now that its working, lets enable the security controls we disabled earlier
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Changing the verification mode to none

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Save and close the elasticsearch.yml file. Restart the elasticsearch.service:

```bash
$ sudo systemctl restart elasticsearch.service
```
Now, lets generate passwords for elastic search

```bash
$ cd /usr/share/elasticsearch/bin
$ sudo ./elasticsearch-reset-password -u elastic
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Do the same for the kibana_system user. 

```bash
$ sudo ./elasticsearch-reset-password -u kibana_system
```
Generating Kibana keys

```bash
$ cd /usr/share/kibana/bin/
$ sudo ./kibana-encryption-keys generate -q
```
Adding the keys to the config file

```bash
$ sudo nano /etc/kibana/kibana.yml
```

Uncommenting the config file (port and the address)
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>


Further changes to the config file

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Start the Kibana service and check the service status

```bash
$ sudo systemctl start kibana.service
$ sudo systemctl status kibana.service
```

We can now access the elastic login page

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Creating users and assigning roles

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Logging in from the newly created user

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

<h3>Installing & Configuring Filebeat & Auditbeat</h3>

Jump back over to your Suricata host to download and install the Public Signing key:

```bash
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Save the repository definition to /etc/apt/sources.list.d/elastic-8.x.list

```bash
$ echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
```

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Next, we will configure Filebeat to connect to Elasticsearch and Kibana. 

```bash
$ sudo nano /etc/filebeat/filebeat.yml
```

Adding the correct address and passwords to connect the Suricata machine to the machine that’s running the ELK stack.
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Configuring the filebeat modules

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Connecting filebeat

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Setting up filebeat
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Starting the filebeat service

```bash
$ sudo systemctl start filebeat.service
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Install Auditbeat with apt.

```bash
$ sudo apt-get update && sudo apt-get install auditbeat
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Configuring the auditbeat

```bash
$ sudo nano /etc/auditbeat/auditbeat.yml
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Start auditbeat

```bash
$ sudo systemctl start auditbeat.service 
```
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

<h2>Viewing Logs in Kibana </h2>
Log back into Kibana using the account you created</br>
Success!! We can now visualize our Suricata logs with Kibana.

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>


