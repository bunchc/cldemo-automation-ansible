Linux TC Demo
=======================
This demo demonstrates how to use traffic control (tc) to set both IP traffic limits and minimum guarantees.

This demo is written for the [cldemo-vagrant](https://github.com/cumulusnetworks/cldemo-vagrant) reference topology and applies the reference BGP unnumbered configuration from [cldemo-config-routing](https://github.com/cumulusnetworks/cldemo-config-routing).

On Server02, 

Quickstart: Run the demo
------------------------
(This assumes you are running Ansible 1.9.4 and Vagrant 1.8.4 on your host.)

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch leaf01 leaf02 spine01 spine02 server01 server02
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    sudo apt-get install software-properties-common -y
    sudo apt-add-repository ppa:ansible/ansible -y
    sudo apt-get update
    sudo apt-get install ansible -qy
    git clone https://github.com/bunchc/linux-tc-demo
    cd linux-tc-demo
    ansible-playbook run-demo.yml
    ssh server01
    tmuxinator start iperf


Topology
--------
This demo runs on a spine-leaf topology with two single-attached hosts. Each device's management interface is connected to an out-of-band management switch and bridged with the out-of-band management server from which we run Ansible.

             +------------+       +------------+
             | spine01    |       | spine02    |
             |            |       |            |
             +------------+       +------------+
             swp1 |    swp2 \   / swp1    | swp2
                  |           X           |
            swp51 |   swp52 /   \ swp51   | swp52
             +------------+       +------------+
             | leaf01     |       | leaf02     |
             |            |       |            |
             +------------+       +------------+
             swp1 |                       | swp2
                  |                       |
             eth1 |                       | eth2
             +------------+       +------------+
             | server01   |       | server02   |
             |            |       |            |
             +------------+       +------------+


Setting up the Infrastructure
-----------------------------
This lab configures HTB or hierarchical token bucket as the preferred packet scheduler on ```server02```. It then creates two traffic classes, one for port 8000 and one for port 8080. From there it creates three filters. The first two are using the u32 syntax of tc to filter on inbound and outbound ports. The third filter identifies traffic classified by IPTABLES.

