---
tags: Security of systems and networks
---
:::success
# SSN Lab 6 - Intrusion Detection Systems / Intrusion Prevention Systems
Name: Ivan Okhotnikov
:::


## Task 0 - Installation:

:::success
### References:
1. [How to install Snort](http://sublimerobots.com/2017/06/snort-ips-with-nfq-routing-on-ubuntu/)
:::
:::info

<center>

![](https://i.imgur.com/dI6CvTm.png)
Figure 1: My topology
</center>

In order for clients to have Internet access and communicate with each other, you need to enter the following commands:
```
echo 'net.ipv4.ip_forward = 1' >>/etc/sysctl.conf; sysctl -p
echo 'net.ipv4.conf.all.forwarding = 1' >>/etc/sysctl.conf; sysctl -p
sudo iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
```

Now install the necessary packages:
```
sudo apt update
sudo apt-get install -y build-essential libpcap-dev libpcre3-dev libdumbnet-dev bison flex zlib1g-dev liblzma-dev openssl libssl-dev make libnetfilter-queue-dev gcc libluajit-5.1-dev libnghttp2-dev libdnet autoconf libtool
```

After installing all the necessary packages, I will proceed to installing Snort and DAQ for it
But before I start, I will create a folder in the home directory and go to it

```
mkdir ~/snort_src
cd ~/snort_src
```

### DAQ

I download the DAQ archive from the official website and unpack it
```
wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
tar -xvzf daq-2.0.7.tar.gz
cd daq-2.0.7
```

Next, I configure the sources and install:

```
./configure
make
sudo make install
```

DAQ is installed, now I can proceed to installing Snort

### Snort
#### Installation
I do the same thing, with the difference that I will specify the `--enable-sourcefire` flag when configuring the sources (I do this to enable specific Sourcefire build parameters, including `--enable-perfprofiling` and `--enable-ppm`)
```
cd ~/snort_src
wget https://www.snort.org/downloads/snort/snort-2.9.18.1.tar.gz
tar -xvzf snort-2.9.18.1.tar.gz
cd snort-2.9.18.1
./configure --enable-sourcefire
make
sudo make install
```
After installation, be sure to update the shared libraries:
```
sudo ldconfig
```

I can check that everything is installed correctly using the command (figure 2):
```
snort -V
```

<center>

![](https://i.imgur.com/Nx8JdVE.png)
Figure 2: Checking the snort installation
</center>


To work correctly after installation, I will create a user `snort`:

```
sudo groupadd snort
sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
```

Creating a file structure for the program:
```
# Create the Snort directories:
sudo mkdir /etc/snort
sudo mkdir /etc/snort/rules
sudo mkdir /etc/snort/rules/iplists
sudo mkdir /etc/snort/preproc_rules
sudo mkdir /usr/local/lib/snort_dynamicrules
sudo mkdir /etc/snort/so_rules
 
# Create some files that stores rules and ip lists
sudo touch /etc/snort/rules/iplists/black_list.rules
sudo touch /etc/snort/rules/iplists/white_list.rules
sudo touch /etc/snort/rules/local.rules
sudo touch /etc/snort/sid-msg.map
 
# Create our logging directories:
sudo mkdir /var/log/snort
sudo mkdir /var/log/snort/archived_logs
 
# Adjust permissions:
sudo chmod -R 5775 /etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /var/log/snort/archived_logs
sudo chmod -R 5775 /etc/snort/so_rules
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
 
# Change Ownership on folders:
sudo chown -R snort:snort /etc/snort
sudo chown -R snort:snort /var/log/snort
sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
```

And I will transfer the standard configuration files:

```
cd ~/snort_src/snort-2.9.18.1/etc/
sudo cp *.conf* /etc/snort
sudo cp *.map /etc/snort
sudo cp *.dtd /etc/snort
 
cd ~/snort_src/snort-2.9.18.1/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor/
sudo cp * /usr/local/lib/snort_dynamicpreprocessor/
```

Then I would really like to disable all the rules in the Snort configuration file so that I can include the necessary rule files myself in the future:

```
sudo sed -i 's/include \$RULE\_PATH/#include \$RULE\_PATH/' /etc/snort/snort.conf
```
#### Configuration
It's time to start editing the configuration file `/etc/snort/snort.conf`

```
sudo nano /etc/snort/snort.conf
```

In it, I will specify the server subnet in the `HOME_NET` variable:

```
ipvar HOME_NET 10.10.5.0/24
```

And the external subnet will be the inversion of the home subnet (in other words, everything that is not the home directory)

```
ipvar EXTERNAL_NET !$HOME_NET
```

It will also be necessary to bring the variables below (since they now have a relative path) to the following form:

```
var RULE_PATH /etc/snort/rules              
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
 
var WHITE_LIST_PATH /etc/snort/rules/iplists
var BLACK_LIST_PATH /etc/snort/rules/iplists
```

To make it easier to work with the rules, I will comment out the line:

```
include $RULE_PATH/local.rules
```

I will turn on daq in nfq mode and set the number of queues to 0 (so that all packets pass only through Snort)
```
config daq: nfq
config daq_mode: inline
config daq_var: queue=0
```
Also I will include iptables rules for this:

```
sudo iptables --append FORWARD --jump NFQUEUE --queue-num 0
```

:::
<!--         ens4:
            addresses:
              - 10.10.5.1/24
        ens5:
            addresses: 
              - 10.20.5.1/24
 -->






## Task 1

:::warning
- Create a policy/rule to block all incoming ping packets.
- Verify that newly created rule/policy can correclty detect and block incoming ping packets from attacker machine.
:::

## Implementation
:::info

I will make the rules in the previously included file in the configuration file:

```
sudo nano /etc/snort/rules/local.rules
```

And I made the following rule into it:

```
drop icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"PING detected"; sid:2; rev:1;)
```

This rule means the removal of any icmp packets to the home subnet (and a request to write a message that I previously indicated in the logs)

After that, I start pinging my host and see messages in the program output, as well as the unavailability of the host in the ping command window (figure 3)
```
sudo snort -c /etc/snort/snort.conf -A console -Q
```

<center>

![](https://i.imgur.com/gknSXJZ.png)
Figure 3: Ping command on the attacking machine and output in the snort program
</center>
:::

## Task 2
:::warning
- Write 5 more rules/policies of your own that can detect different attacks (possible attacks).
- Show the new 5 rules and explain how the attacks would work and how the rule/policy would be able to detect it.
- Describe how you triggered the alert for all newly created rules.
- Attach the alert itself for each given rule/policy to the report in readable text.
:::

## Implementation
:::info

To include all the newly added rules, I just added them to the file `/etc/snort/rules/local.rules`

Rule 1: deleting all potential requests sent to the web server in order to cause a denial of service to clients

```
drop tcp $EXTERNAL_NET any -> $HOME_NET 80 (flags: S; msg:"Possible DoS Attack on a web server"; flow:stateless; sid:2; detection_filter:track by_src, count 1000, seconds 60;)
```

To cause such an attack, I will use the `hping3` utility with an indication that there will be a flood on the 80 port of the server `10.10.5.2`

```
sudo hping3 --flood -S -p 80 10.10.5.2
```

The output from Snort and the attack itself is shown in the screenshot below

<center>

![](https://i.imgur.com/oyY1U4I.png)
Figure 4: Dos attack on the server
</center>

---

Rule 2: detecting requests to a web server without using a browser (using the wget command)

```
alert tcp $EXTERNAL_NET any -> $HOME_NET 80 (msg:"Attempt to send a request without a browser using wget"; flow:to_server,established; content:"Wget"; http_header; depth:500; sid:3;)
```

Command to check the rule:
```
wget 10.10.5.2
```

<center>
    
![](https://i.imgur.com/9nLpXlO.png)
Figure 5: Output from Snort and the request itself
</center>

---

Rule 3: detecting a brute force attack on an `ssh server`

```
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"Bruteforce attack on ssh server"; detection_filter:track by_src, count 2000, seconds 60;sid:4;)
```

To reproduce this attack, I can use the nmap command to run the `ssh-brute` script
```
sudo nmap -p 22 --script ssh-brute 10.10.5.2
```
<center>
    
![](https://i.imgur.com/zVKHFju.png)
Figure 6: Withdrawal from Snort and attack
</center>

---

Rule 4: detection of a `Ping of death` attack, the essence of which is the size of a long request

```
alert ip $EXTERNAL_NET any -> $HOME_NET any (dsize:>500; msg: "Ping of death detected"; sid:5;)
```

I can call this command using the `-s` flag of the ping command
I used the `-f` flag to call the flood
```
ping 10.10.5.2 -s 1000 -f
```

<center>
    
![](https://i.imgur.com/LetpXh5.png)
Figure 7: Withdrawal from Snort and attack
</center>

---

Rule 5: detection of port scanning by the NMAP command, since this program sends requests with a zero size

```
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"Attempt to scan the ping sweep using NMAP"; dsize:0; sid:6;)
```

Command to check the rule:
```
nmap 10.10.5.2
```
<center>

![](https://i.imgur.com/7iKFyKk.png)
Figure 8: Output from Snort and port analysis from Nmap
</center>
:::

## Task 3 - Create one rule/policy which would be triggered if a server vm / IDS/IPS vm would browse to microsoft.com website.

## Implementation
:::info
To do this, I will create a rule specifying `content:"microsoft.com "`
```
alert tcp $HOME_NET any -> any any (msg:"Microsoft.com site detected"; content:"microsoft.com"; sid:5; rev:1;)
```
To check this rule, I will download the site page `microsoft.com` using the `wget` command
<center>
    
![](https://i.imgur.com/DXQD92b.png)
Figure 10: Output from Snort and the wget command from the attacker
</center>
:::

## Task 4 - Answer the following questions:

:::warning
- What is a zero-day attack?
- Can IDS/IPS detect zero-day attack? if yes why? if no why?
- Given a network that has 1 million connections daily where 0.1% are attacks. If the IDS has a true positive rate of 95% what false positive alarm rate do we need to achieve to ensure the probability of an attack, given an alarm is 95%?
:::

## Implementation
:::info
> - What is a zero-day attack?
> - Can IDS/IPS detect zero-day attack? if yes why? if no why?

The zero-day attack is an attack that is still known only to attackers and developers do not have time to fix it.
It can only be detected if it is connected to networks.
IDS/IPS can see anomalies in traffic and report it

> - Given a network that has 1 million connections daily where 0.1% are attacks. If the IDS has a true positive rate of 95% what false positive alarm rate do we need to achieve to ensure the probability of an attack, given an alarm is 95%?

Since there are 1 million connections every day,
the number of attacks that make up 0.1% will be only `1000000*0.001 = 1000` attacks daily

The number of true positive positives is `1000*0.95 = 950`

The frequency of false positives will be equal to: `F(y) = (1000000 - 1000) * y /100`

To calculate false positive positives:
`FT = 950 / (950 + F(y))`
To find `y`, it is necessary to equate `FT` to `0.95` and compensate for the internal expression by subtracting from `0.100`

`0,95 = 0,100 - 950 / (950 + (999000*y/100))`
`0,05 = 950 / (950 + 9990y)`
`0,05 * (950 + 9990y) = 950`
`950+9990y = 950 / 0.05`
`9990y = 19000 - 950`
`9990y = 18050`
`y = 1.806`

After simplifying the expression, it turns out that **y** = `1.806%`


:::
