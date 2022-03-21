# vpn-wireguard

### Updates packages:
#### `apt update && apt upgrade -y`

### Install wireguard:
#### `apt install -y wireguard`

### Generates server keys:
#### `wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey`

###  Modifies File Permissions:
#### `chmod 600 /etc/wireguard/privatekey`

### Check network interface:
#### `ip a`

Most likely you have `eth0`, but probably another one as `ens3` or other. This name of network interface we'll use in the config `/etc/wireguard/wg0.conf`, which we should create before.

### Put into `/etc/wireguard/wg0.conf`:
```
[Interface]
PrivateKey = <private_key>
Address = 10.0.0.1/24
ListenPort = 51830
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o <net_interface> -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o <net_interface> -j MASQUERADE
```

- Replace `<net_interface>` to your name of network interface in the lines `PostUp` and `PostDown`
- Replace `<private_key>` by key from `/etc/wireguard/privatekey`

### Set up IP forwarding:
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### Enable systemd daemon with wireguard:
```
systemctl enable wg-quick@wg0.service
systemctl start wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```

### Generates clients keys:
#### `wg genkey | tee /etc/wireguard/yourname_privatekey | wg pubkey | tee /etc/wireguard/yourname_publickey`

### Add to server config `/etc/wireguard/wg0.conf`:
```
[Peer]
PublicKey = <yourname_publickey>
AllowedIPs = 10.0.0.2/32
```

Replace `<yourname_publickey>` by your key from `/etc/wireguard/yourname_publickey`

### Restart systemd service with wireguard:
```
systemctl restart wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```

### Create new client config file on your local PC - `yourname.conf`:
```
[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = 10.0.0.2/32
DNS = 1.1.1.1, 8.8.8.8

[Peer]
PublicKey = <SERVER-PUBKEY>
Endpoint = <SERVER-IP>:51830
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```

- Replace `<CLIENT-PRIVATE-KEY>` by client private key `/etc/wireguard/yourname_privatekey` in server.
- Replace `<SERVER-PUBKEY>` by public server key `/etc/wireguard/publickey` in server.
- Replace `<SERVER-IP>` by your IP server.
