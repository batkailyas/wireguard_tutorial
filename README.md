# How to setup wireguard on VPC

First of all you need to SSH to your server

Update the OS and reboot it:

`dnf update -y --refresh`

`reboot`

Then install required packages:

`dnf install wireguard-tools vim firewalld -y`

Now it's time to generate private and public keys for server and initial client:
Create hidden folder in home directory:

`mkdir ~/.wireguard`

`wg genkey | tee .server_private_key | wg pubkey > server_public_key`

`wg genkey | tee .client_private_key | wg pubkey > client_public_key`

`chmod 600 .*_private_key`

As you noticed I'm making private keys as hidden files also. 


Let's put server private key and client pub key into config without revealing them:

`cat .server_private_key > /etc/wireguard/wg.conf`

`cat client_public_key`

Now it's time to edit our config file
`vim /etc/wireguard/wg.conf`

Copy this template and put your keys:

```[Interface]
Address = 192.168.123.1/24
PrivateKey = SERVER_PRIVATE_KEY
ListenPort = 51820

# Client #1
[Peer]
AllowedIPs = 192.168.123.2/24
PublicKey = CLIENT_PUBLIC_KEY
```

As you finished with config let's enable and configure firewall:

`systemctl enable firewalld`

`systemctl start firewalld`

Use this command to check existing firewall rules:

`firewall-cmd --list-all`

Use following commands to open port and add masquerade:

`firewall-cmd --permanent --add-rich-rule='rule family=ipv4 port port=51820 protocol=udp accept'`

`firewall-cmd --permanent --add-rich-rule='rule family=ipv4 masquerade'`

Reload firewall to apply new changes:

`firewall-cmd --reload`

And check new firewall rules:

`firewall-cmd --list-all`

Now you are ready to start wireguard:

`systemctl start wg-quick@wg`

`systemctl enable wg-quick@wg`


**How to install and configure client**

If you're using macOS - go to AppStore and download WireGuard

If you're using Fedora use following command:
`sudo dnf install wireguard-tools`

If you're using Ubuntu go reinstall it to something else :)

When you've installed WireGuard client it's time to configure it. 

Use this template to configure client:

```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 192.168.123.2/32

[Peer]
PublicKey = SERVER_PUBLIC_KEY
AllowedIPs = 0.0.0.0/0
Endpoint = SERVER_IP_ADDRESS:PORT
PersistentKeepalive = 21
```
Put client private key, server pub key and server ip and port instead of temporary variables in the config above.
