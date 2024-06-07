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
<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Next, update the repository we just added:
```bash
$ sudo apt-get update
```

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Now, install Suricata using the following command:

```bash
$ sudo apt-get install suricata
```
Just like that, you now have Suricata installed on your Ubuntu machine. Now, this is where the fun stuff comes into play. The next steps will show how to configure Suricata.

First, enable the suricata.service to start at boot. This will allow Suricata to run whenever the machine restarts:

```bash
$ sudo systemctl enable suricata.service
```

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Run the following command to verify the Suricata service is running:

```bash
$ sudo systemctl status suricata.service
``` 

<div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

As you can see, Suricata is currently running in IDS (af-packet) mode

The ifconfig command will show all the available interfaces
To configure the suricata.yaml file, note that Suricata inspects traffic on eth0 by default. If your system uses a different interface, update this value accordingly.
```bash
$  ifconfig 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Another alternative command is to show your default network interface is the following:
```bash
ip -p -j route show default 
``` 
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Edit the suricata.yaml configuration file.	
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Configuring the correct interface in the config file (ens33)
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Adding rules to config file
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Killing the process and restaring
```bash
$ sudo kill -usr2 $(pidof suricata)
$ sudo service suricata restart
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Updating suricata
```bash
$ sudo suricata-update
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Updating sources
```bash
$ sudo suricata-update --no-check-certificate update-sources 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Listing the rule providers
```bash
$ sudo suricata-update list-sources
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

fetch the et/open rule
```bash
$ sudo suricata-update enable-source et/open
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Run the suricata-update command again to load et/open ruleset and then run the kill command to update the rules without restarting.
```bash
$ sudo suricata-update 
$ sudo kill -usr2 $(pidof suricata)
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Test Suricata has zero errors once more. 
```bash
$ sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Testing with a curl
```bash
$ curl http://testmynids.org/uid/index.html
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Tailing and looking at the log.

```bash
$ tail /var/log/suricata/fast.log 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Viewing the log in JSON format

```bash
$ sudo apt install jq
$ jq 'select(.alert .signature=="GPL ATTACK_RESPONSE id check returned root")' /var/log/suricata/eve.json
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

<h3>Configuring Suricata to IPS</h3>

Opening the config file to make changes<br>

Find the LISTENMODE=af-packet and comment it out with a #. Then add the following line:
LISTENMODE=nfqueue 

```bash
$ sudo nano /etc/default/suricata
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Restart Suricata
```bash
$ sudo service suricata restart 
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>

Now check the status of the Suricata service. 

```bash
$ sudo service suricata status
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
We will see that Suricata is running in IPS mode now

Open IPv4 rules with the following command:

```bash
$ sudo nano /etc/ufw/before.rules
```
 <div align="center">
  <img src="" alt="logo" width="500" height="500">
</div>
Add the following lines:

-I INPUT -j NFQUEUE 
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

<h1>Integrating Suricata with ELK Stack<h1>

Configuring the elastic config file
