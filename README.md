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

