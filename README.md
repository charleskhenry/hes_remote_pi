# hes_remote_pi

1. Configure the raspberry pi.
   Using Raspberry Pi OS, Debian Bullseye with Security Updates and Desktop 
2. Add the device to you network.
3. SSH into the device
4. Download and isntall VirtualHere
   curl https://raw.githubusercontent.com/virtualhere/script/main/install_server | sudo sh
5. Verify the service is running.
   `systemctl is-active virtualhere`
6. Install a firewall.
   sudo apt-get install ufw
   sudo ufw allow 7575/tcp
   sudo ufw allow 7575/udp
   sudo ufw allow 22/tcp
   sudo ufw allow 5353/upd
   sudo ufw allow 1194/dup
   sudo ufw allow 1194/tcp
   sudo ufw enable
   sudo ufw status
7. Install OpenVPN
   sudo apt-get install openvpn
   mkdir -p ~/openvpn-config
   cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0/* ~/openvpn-config
   cd ~/openvpn-config
   Initialize the configuraiton and generate our keys and certs.
   ./vars
   ./clean-all
   ./build-ca
   ./build-key-server server
   Build keys for client devices
   ./build-key client1
   Generate a Diffie-Hellman Parameters
   ./build-dh
   Generate a HMAC Signature
   openvpn --genkey --secret keys/ta.key
   Create Server Configruation File
   ~/openvpn/server.conf
   
port 1194
proto udp
dev tun
ca keys/ca.crt
cert keys/server.crt
key keys/server.key
dh keys/dh2048.pem
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
tls-auth keys/ta.key 0
cipher AES-256-CBC
comp-lzo
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3

    Start the OpenVPN Server
    sudo systemctl start openvpn@server
https://chat.openai.com/c/a6fea488-e885-4dda-8cdd-3024d39dda3f
    Enable the service to start on boot
    sudo systemctl enable openvpn@server

    Configure IP Forwarding
    sudo nano /etc/sysctl.conf
    
    Uncomment the below line.
net.ipv4.ip_forward=1

   Apply the new file changes
   sudo sysctl -p

Provide the client configuration files (.ovpn) along with the necessary keys and certificates generated weaily to your client devices

Configure the Main Router to port forward the following:
7575 -> pi1 (USB Server)
22 -> pi1 (SSH)
1194 -> pi1 (OpenVPN)

Configure the Main Router to update DDNS.
