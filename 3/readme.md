# Question 3
## Research on IDS/IPS. There are many applications that provide IDS/IPS service. Find one of the opensource service and install it on the VM1 you created previously. Try to set up IDS which block some of the known vulnerabilities signature.
## For IDS/IPS I have used Suricata Application
## commands used
### To install suricata 
- > yum install epel-release yum-plugin-copr
- > yum copr enable @oisf/suricata-6.0
- > yum install suricata
### configuration file located at /etc/suricata/suricata.yaml
- > vi /etc/suricata/suricata.yaml
Edit HOME_NET
HOME_NET variable should include, in most scenarios, the IP address of the monitored interface and all the local networks in use
Capture setting:
af-packet:
    - interface: enp0s3
    - cluster-id: 99
    - cluster-type: cluster_flow
    - defrag: yes
### disable packet offload features on the network interface on which Suricata is listening
- > ethtool -K enp0s3 gro off lro off
### Verify this
- > ethtool -k enp0s3 | grep large
### Adding new sample rules
- > vim /usr/share/suricata/rules/test.rules
Add following line
alert http any any -> any any (msg:"Do not read gossip during work";
content:"Scarlett"; nocase; classtype:policy-violation; sid:1; rev:1;)
Save and exit
### Add rules name in suricata.yaml
- > vim /usr/share/suricata/rules/suricata.yaml
Find rule-files
And under it write 
- > - /usr/share/suricata/rules/test.rules
### fire Suricata in PCAP live mode by executing
- > suricata -D -c /etc/suricata/suricata.yaml -i enp0s3
### fetches the ET Open ruleset
- > suricata-update
### Tail the Suricata alert logs on Suricata host to see what is happening;
- > tail -f /var/log/suricata/fast.log
