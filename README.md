# wg-access-tutorial-oss-europe-2024

## Setup Server Interface

1. Login  
```
ssh root@your-server
```
2. Install wg tools  
```
apt install wireguard-tools
```
3. Check on which interface we'll be forwarding  
```
ip a
```
4. Goto WirGuard folder, create a keypair  
```
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
cat privatekey
```
5. copy the private key to your clipboard  
6. Create the config file  
```
vim wg0.conf
```
7. Enter the file contents with the following format  
```
[Interface]
PrivateKey = (from clipboard)
Address = 10.191.143.x/32 (please replace x with your server number, e.g. "server-122" == 122)
ListenPort = 51820 (optional, is 51820 by default)

PostUp = sysctl -w net.ipv4.ip_forward=1 (adds ip forwarding)
PostUp = iptables -t nat -A POSTROUTING -o <interface from step 3> -j MASQUERADE (will forward to the specified interface, should be eth1)
PostDown = iptables -t nat -D POSTROUTING -o <interface from step 3> -j MASQUERADE (deletes forwarding rule on interface down)

[alternative postup using nftables] PostUp = nft add rule ip nat postrouting oifname "eth1" masquerade
[alternative down using nftables] PostDown = nft delete rule ip nat postrouting oifname "eth1" masquerade
```
8. Create the interface       
```
wg-quick up wg0
```
9. Check on the interface  
```
wg
ip a
```

## Add Peer to Server for your Local Device

1. Create a public/private keypair for your local peer  
**NOTE:** typically you want to generate this on the local device. Private keys should stay on the peer. However, this is more convenient for the tutorial. Feel free to generate locally if you prefer.   
```
cd /etc/wireguard
mkdir peers
cd peers
wg genkey | tee privatekey | wg pubkey > publickey
cat publickey
```
2. copy the public key  
3. modify the server config to include the peer    
```
cd ..
vim wg0.conf
```
4. Enter contents with the following format, below the PostDown command  
```
[Peer]
PublicKey = <copied from step 2>
AllowedIPs = 10.191.143.2/32 (can be any private address you prefer, but should not be the same as the server private IP)
PersistentKeepalive = 25
```
5. Restart the wireguard interface, check to see it includes the new peer  
```
wg-quick down wg0 && wg-quick up wg0
wg
```
6. Copy the public key of the **server** and the private key of the **peer** for the next steps  
```
cat publickey
cat peers/privatekey
```

## Setup WireGuard Connection Locally

1. Install WireGuard: https://www.wireguard.com/install/
2. Create Config file (depends on your OS. This is for Linux. Or just use any text editor)
```
vim wg0.conf
```
3. Add contents to Config File using the following template  
```
[interface]
PrivateKey = <peer private key>
Address = 10.191.143.2/32 (should be whatever address was added to Peer section in Step 4 of previous instructions)
DNS = 10.101.0.6 (this is the DNS server in the private environment)

[Peer]
PublicKey = <server public key>
Endpoint = <server public address>:51820 (the public address will be the same address you used to SSH)
AllowedIPs = 10.191.143.x/32,10.101.0.0/16 (this will be the server address used in Step 7 of the first section, and the VPC CIDR, which is 10.101.0.0/16)
PersistentKeepalive = 25
```
4. Start the interface. This is the command for Linux. If using GUI, upload and activate the config file. Or use whatever instructions for your OS  
```
wg-quick up ./wg0.conf
```
## Check Access from Local

These instructions are for Linux. For Windows or other OS, check GUI and just check website access

1. Check to see if interface exists and you have a handshake with server
```
ip a
wg
```
2. Check to see if you can access resources  
```
ping 10.101.0.3
ping webserver.demo.com
```
3. Goto webserver.demo.com and fileserver.demo.com in your browser  

## Alternative Access w/ Pre-Generated Config File

1. Go to https://drive.google.com/drive/u/0/folders/1HyBd6-vJYQsWkBsrD6ZgprhTI5LRgVLs
2. Download and apply a config file (config files will only work on 1 device at a time)

## Got Time? Build a Peer-to-Peer Network with your Peers!

1. Add your **server** public key here: https://docs.google.com/spreadsheets/d/112ir8ZVfqklE61bmJx7CZVkGIjJTPeGd3di6SPMiBoU/edit?usp=sharing   
2. Wait for keys to appear from your group in the doc  
    - Alternatively, talk to your neighbors. Trade keys with them.  
4. Add your peers as peers to your **server** config file  
5. For each peer, add a section to your config file as follows:  
```
[Peer]
PublicKey = <peer public key>
Endpoint = <peer ip address>:51820
AllowedIPs = <peer private address>/32
PersistentKeepalive = 25
```
5. Restart the interface  
```
wg-quick down wg0 && wg-quick up wg0
```
6. Note this will only work once a peer **has also added you**. Check your WireGuard interface to see if you have a connection. If there is a handshake, try pinging.  
```
wg
ping <peer private address>
```
### Extra Points: Run a private webserver, write a secret message
1. Install docker on your server
```
apt install docker.io
```
2. Run a webserver on your wireguard interface
```
docker run -d -e MESSAGE="YOUR SECRET MESSAGE HERE" -p <your server private wg address>:8080:80 nginx:alpine sh -c "echo \$MESSAGE > /usr/share/nginx/html/index.html && nginx -g 'daemon off;'"
```
3. check other servers in your P2P network for their secret messages
```
curl <peer private ip address>:8080
```
