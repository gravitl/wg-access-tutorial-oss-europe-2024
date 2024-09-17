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
[interface]
PrivateKey = (from clipboard)
Address = 10.191.143.1/32 (can be any private address you'd like)
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
