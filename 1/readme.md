# Qusttion No. 1
## Setup a VPN server in one vm. You can use openvpn for this purpose.
- ### You should have two network interfaces one for wan and another for lan. You should set up a vpn server which listens on the WAN interface and provides a LAN interface subnets ip address to the client which connects using openvpn client
- ### You should create certificates files for both server and client to connect to server and export client certificates to the client vm..
	## commands used
	### install dependencies and openvpn
	- > sudo yum install -y epel-release
	- > sudo yum install -y openvpn wget
	### install Easy-RSA
	- > wget -O /tmp/easyrsa https://github.com/OpenVPN/easy-rsa-old/archive/2.3.3.tar.gz
	- > tar xfz /tmp/easyrsa
        ### Creating a sub-directory under /etc/openvpn and extracting EasyRSA files over here.
	- > sudo mkdir /etc/openvpn/easy-rsa
	- > sudo cp -rf easy-rsa-old-2.3.3/easy-rsa/2.0/* /etc/openvpn/easy-rsa
        ### Changing directory’s owner to non-root sudo user
	- > sudo chown bibek /etc/openvpn/easy-rsa/
	### Configuring OpenVPN
	- > sudo cp /usr/share/doc/openvpn-2.4.11/sample/sample-config-files/server.conf /etc/openvpn/
	- > sudo nano /etc/openvpn/server.conf
	### Creating keys directory where Easy-RSA will store any keys and certs we generate
	- > sudo mkdir /etc/openvpn/easy-rsa/keys
	### Default certificate variables are set in vars file in /etc/openvpn/easy-rsa
	- > sudo nano /etc/openvpn/easy-rsa/vars
	### To start generating keys, move to easy-rsa directory and source in the new variables
	- > cd /etc/openvpn/easy-rsa
	- > source ./vars
	### Clean any keys and certificates already in the folder
	- > ./clean-all
	### Build certificate authority. 
	- > ./build-ca
	### Creating key and certificate for the server
	- > ./build-key-server server 
	### Creating diffie helmen key exchange file
	- > ./build-dh
	### copy the server keys and certificates from keys directory to openvpn directory
	- > cd /etc/openvpn/easy-rsa/keys
	- > sudo cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
        ### Generating client keys
	- > cd /etc/openvpn/easy-rsa
	- > ./build-key client
        ### Copy versioned OpenSSL configuration file to versionless name to load configuration
	- > cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
	### Giving instructions to OpenVPN about where to send incoming web traffic (ROUTING)
	- > sudo firewall-cmd --zone=external --add-service openvpn --permanent
	- > sudo firewall-cmd --permanent --add-masquerade
	- > sudo firewall-cmd --query-masquerade
	### Creating variable SHARK which will represent the primary network interface
	- > SHARK=$(ip route get 8.8.8.8 | awk 'NR==1 {print $(NF-2)}')
	### Using SHARK variable to permanently add the routing rule to our subnet
	- > sudo firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.10.1.0/24 -o $SHARK -j MASQUERADE
	- > sudo firewall-cmd --reload
	- > sudo systemctl restart network
	### Now we are ready to start openvpn service
	- > sudo systemctl -f enable openvpn@server.service
	- > sudo systemctl start openvpn@server.service
	- > sudo systemctl status openvpn@server.service
	### To transfer client certificate to client machine, I used rsync command
	- > cd /etc/openvpn/easy-rsa/keys
	- > sudo rsync ca.crt client.crt client.key ../../myvpn bibek@192.168.1.142:/home/bibek/openclient




