# Question 2
## Create another vm with installing openvpn client and you should create an openvpn client connect file(with extension .ovpn) providing the certificates. You should be able to connect to the vpn server and get an ip address from the LAN subnet that you assigned in the first vm.
## commands used
### created client.ovpn in the same directory where I have transferred the client keys
- > sudo nano client.ovpn
added following lines
client
tls-client
ca ca.crt
cert client.crt
key client.key
tls-crypt myvpn.tlsauth
remote-cert-eku "TLS Web Server Authentication"
proto tcp
remote 192.168.1.139 1194 udp
dev tun
topology subnet
pull
user nobody
group nobody
auth-user-pass
### enable user-password authentication
- > sudo vi /etc/pam.d/openvpn
And add the following lines
auth    required        pam_unix.so     shadow  nodelay
account required        pam_unix.so
### install openvpn-client to ubuntu server
- > sudo apt install -y openvpn
### connect to openvpn server
- > sudo openvpn --config client.ovpn
