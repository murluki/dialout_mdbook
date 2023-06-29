# Routing Traffic via Nym

![Alternative text](../img/1.jpg)

Our product is a wireless AP that has pi-hole on it and nym-gateway to access nym mixnet over nym-socks5-client. All traffic from AP’s guests (Telegram, Signal, Monero wallet, Metamask wallet and Keplr wallet) should be routed through the nym-socks5-proxy running at localhost:1080 on the WiFi AP. Other traffic would be routed over the regular default gateway on the wifi AP. We are using Raspberry Pi OS.


## System Setup:
Given our Raspberry Pi 4 with the 64-bit Raspberry Pi OS (arch: aarch64-unknown-linux-gnu), our system setup will proceed as follows:

## 2.1. Initial Setup:

Download and flash the Raspberry Pi OS onto a microSD card using software like balenaEtcher.
Since we're running the Pi headless and without GUI, we'll create a file named 'ssh' in the boot partition of the SD card to enable SSH access.
Insert the microSD card into the Raspberry Pi, connect it to the network, and power it up.
## 2.2. Secure Shell (SSH) Access:

Find the Raspberry Pi's IP address on your network.
Use an SSH client like PuTTY (for Windows) or the terminal (for Linux/Mac) to access the Raspberry Pi. The default credentials are:
username: pi
password: raspberry
## 2.3. System Update:

Once logged in, it's a good practice to update the system. Run the following commands:
bash
Copy code
sudo apt-get update
sudo apt-get upgrade
## 2.4. Configuring Network Settings:

Configure the network settings to match your local network environment.
## 2.5. Disable GUI (Optional):

Since we want to run headless and without GUI, use the command sudo raspi-config to navigate to 'System Options' > 'Boot / Auto Login' > 'Console Autologin' or 'Console'.
With these steps, our Raspberry Pi 4 should be ready for the next stages of our project.

After setting up the system, the next step is to configure the Raspberry Pi as a wireless access point. We will be using hostapd for this purpose, along with dnsmasq to handle DHCP and DNS services for devices connecting to the access point.
write 3. and 4.

Configuring Raspberry Pi as a Wireless Access Point:
## 3.1. Install Necessary Packages:

Begin by installing hostapd and dnsmasq, which will act as our access point and DHCP/DNS server respectively.
bash
Copy code
sudo apt-get install hostapd dnsmasq
## 3.2. Configure hostapd:

Edit the hostapd configuration file:
bash
Copy code
sudo nano /etc/hostapd/hostapd.conf
Add the following (substitute your preferred network name and password):
bash
Copy code
interface=wlan0
driver=nl80211
ssid=Your_Network_Name
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=Your_Password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

## 3.3. Enable and Start hostapd and dnsmasq:

Now, let's enable and start both services:
bash
Copy code
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
Installing and Configuring Pi-hole:

## 4.1. Install Pi-hole:

To install Pi-hole, we'll use the automated script provided by the Pi-hole project:
bash
Copy code
curl -sSL https://install.pi-hole.net | bash
Follow the on-screen instructions to complete the setup.
4.2. Configure Pi-hole:

After the installation, access the Pi-hole admin interface via a web browser by entering the Raspberry Pi's IP address followed by '/admin'.
Configure the settings as per your requirements. Ensure that the Pi-hole is set to listen on the 'wlan0' interface (or whichever interface you're using for your access point).


With the Raspberry Pi now functioning as a Wireless Access Point with Pi-hole running on it, our next step is to set up the Nym Mixnet Gateway and configure the traffic routing.
Help me write the next section. Here is some more info for you:

nym-gateway and nym-socks5-client are installed.  
```
## First set the vars for gateway and sp
##
## NYM_GATEWAY="/home/nym/.nym/gateway.lst"
## NYM_PROVIDER="/home/nym/.nym/sp.lst"
## FIREWALL MUST ALLOW 
## PORTS 1789,1790,8000,9000,22 on TCP for this to work!
## Then run the socks5 after init
#./nym-socks5-client run --gateway $NYM_GATEWAY --provider $NYM_PROVIDER 
## 
##
## Both things would eventually run as a service files like following example:
##

[Unit]
Description=nym mixnode service
After=network.target

[Service]
Type=simple
User=nym
LimitNOFILE=65536
ExecStart=/home/nym/nym-gateway run --id dialout
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target


## SNIFF THE TRAFFIC in multiple terminal windows to show how this is going through -p 1080
## and how other stuff goes through ANY except 108”
```
Setting up Nym Mixnet Gateway:
## 5.1. Set Up Nym Mixnet Gateway Variables:

We need to set the variables for the gateway and service provider. These can be added to your bash profile or set in your current shell:
bash
Copy code
export NYM_GATEWAY="/home/nym/.nym/gateway.lst"
export NYM_PROVIDER="/home/nym/.nym/sp.lst"
## 5.2. Ensure Firewall Rules:

Your firewall must allow traffic on ports 1789, 1790, 8000, 9000, and 22 for TCP. Adjust your firewall settings accordingly.
## 5.3. Run the Nym Socks5 Client:

After initializing, run the socks5 client:
bash
Copy code
./nym-socks5-client run --gateway $NYM_GATEWAY --provider $NYM_PROVIDER
## 5.4. Configure Nym Services to Run Persistently:

For both the nym-gateway and nym-socks5-client, we should set them up to run as services using systemd. An example service file is given above for the nym-gateway. Create similar service files for the nym-socks5-client.
Configuring Traffic Routing:
## 6.1. Sniffing Traffic:

To verify that traffic is being routed correctly, you can sniff the traffic in multiple terminal windows. Tools like tcpdump or Wireshark can help monitor traffic.
Remember, in our setup, traffic from specific applications (Telegram, Signal, Monero wallet, Metamask wallet, Keplr wallet) should be routed through the nym-socks5-proxy running at localhost:1080. Other traffic should be routed through the regular default gateway.


Once we have the Nym services running and the traffic routing verified, our next step is to conduct a full system test to ensure that everything works as expected. Additionally, we will be looking at future enhancements and establishing a roadmap for our project.
Write next task


Testing and Verification:
## 7.1. Test Connectivity:

Test the connection to your WiFi access point and verify that you're able to connect and access the internet.
## 7.2. Verify Pi-hole Functionality:

To ensure Pi-hole is working, try accessing a known ad-serving website from a device connected to the access point. You should notice that ads are being blocked.
## 7.3. Test Nym Functionality:

Verify that the specific applications (Telegram, Signal, Monero wallet, Metamask wallet, Keplr wallet) are successfully communicating through the Nym network. This can be done by using network monitoring tools and observing that this traffic is going through the nym-socks5-proxy at localhost:1080.
## 7.4. Test Default Traffic Routing:

Test other internet activities and confirm they're routed through the default gateway on the WiFi AP, not through the Nym network.
Future Enhancements and Roadmap:
## 8.1. Security Enhancements:

Review security measures and make improvements where necessary. Consider implementing a VPN or additional firewall rules for enhanced security.
## 8.2. Improve Nym Integration:

Explore ways to better integrate Nym into our setup, perhaps by automating certain aspects of the setup process.
## 8.3. User Interface Improvements:

Given that our system is headless, consider developing a web-based interface to allow easier control and configuration of the system.
## 8.4. Performance Tuning:

As the system usage increases, performance tuning might be necessary. Monitor the system and identify areas that could benefit from performance optimization.
Once the testing and verification phase is complete, and we've laid out our future roadmap, we can consider our initial setup complete. Our final step would be ongoing system monitoring and implementing the enhancements outlined in the roadmap.
point 7.3 - please provide a script that could do that routing

To achieve the desired traffic routing, we need to use iptables to create rules that direct certain traffic to go through the Nym socks5 proxy. Here's a general outline of a script that could accomplish this:

bash
Copy code
#!/bin/bash

Define applications that should use the nym-socks5-proxy
apps=("telegram" "signal" "monero" "metamask" "keplr")

Set local variables
proxy_address="127.0.0.1"
proxy_port="1080"

Flush all existing iptables rules
iptables -F
iptables -t nat -F

Loop over each app
for app in "${apps[@]}"
do
   Get the UID of the application
   app_uid=$(id -u $app)

   Add rule for this app's traffic to go through nym-socks5-proxy
   iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner $app_uid -j DNAT --to-destination $proxy_address:$proxy_port

   Add rule for other traffic to go through the default gateway
   iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner $app_uid -j ACCEPT
done

Ensure the changes persist across reboots
iptables-save > /etc/iptables/rules.v4
Please note, this script assumes that the applications are running under their respective user names and that these user names match the strings in the 'apps' array. If this is not the case, you'll need to adjust the 'apps' array to contain the correct user names.

Also, this script requires the 'iptables-persistent' package to be installed in order to save the iptables rules across reboots. You can install it by running sudo apt-get install iptables-persistent.


With this script, we've established the traffic routing needed for our setup. The next step is to test this routing by following the steps outlined in 7.1 to 7.4 of our plan. Additionally, we should begin to think about the future enhancements and roadmap for our project.
the apps are running on guests system not on the Wifi AP! It should do the similar as Pi-hole does with ADs but instead it routes it through socks5 proxy running port 1080.

I see, I misunderstood the setup; I apologize for the confusion. The iptables method I described would work for apps running on the same machine where the iptables rules are being applied. For the scenario where the apps are running on devices connected to the Raspberry Pi access point, we need to use a different approach.

We will need to install and configure a transparent proxy (like redsocks) on the Raspberry Pi to reroute specific traffic to the Nym socks5 proxy. Here's how we can do this:

Install redsocks:
bash
Copy code
sudo apt-get install redsocks
Configure redsocks:
Edit the redsocks configuration file (/etc/redsocks.conf), for example:

bash
Copy code
base {
    log_debug = off;
    log_info = on;
    daemon = on;
    redirector = iptables;
}

redsocks {
    local_ip = 0.0.0.0;
    local_port = 12345;
    ip = 127.0.0.1;
    port = 1080;
    type = socks5;
}

This configuration file tells redsocks to listen on port 12345 and to redirect traffic to the Nym socks5 proxy running on localhost port 1080.

Configure iptables:
Next, we need to configure iptables to redirect traffic from your specified apps to the redsocks port. This will likely involve matching on destination port or IP addresses if the apps use specific ones. Here's an example:

bash
Copy code
Flush all existing rules
sudo iptables -F
sudo iptables -t nat -F

Define the IP ranges or ports for the apps
app_destinations=("IP_or_port1" "IP_or_port2" "IP_or_port3" ...)

Loop over each app IP/port
for dest in "${app_destinations[@]}"
do
    # Add rule for this app's traffic to go through redsocks (and thus the Nym socks5 proxy)
    sudo iptables -t nat -A PREROUTING -p tcp -d $dest -j REDIRECT --to-ports 12345
done

Ensure the changes persist across reboots
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
Please replace "IP_or_port1", "IP_or_port2", etc. with the actual IPs or ports used by your apps.

Restart redsocks service:
bash
Copy code
sudo systemctl restart redsocks

With this setup, traffic from the specified apps should be routed through the Nym socks5 proxy when they are connected to the WiFi access point. The next step is to test this routing to ensure it's working correctly. This involves monitoring the network traffic (as discussed in section 7 of our plan) and verifying that the specified app traffic is indeed going through the proxy.
how to get ranges of IPv4s and IPv6s of hosts for Telegram, Monero wallet, Keplr wallet and Metamask?

There isn't a straightforward way to get the full list of IP addresses used by these applications, mainly due to two reasons:

The list of servers they use can change over time.
These applications could use cloud services for their backend, which means they could technically be served from any IP within the large ranges owned by the cloud provider.
However, one practical approach would be to observe the network traffic generated by these applications and record the IP addresses they communicate with. You can use a tool like Wireshark for this:

Install Wireshark on a device running the applications.
bash
Copy code
sudo apt-get install wireshark
Run Wireshark and start a packet capture.
Open each of the applications (Telegram, Monero wallet, Keplr wallet, and Metamask) and allow them to communicate over the network.
Stop the packet capture and filter the results to show only outbound communication.
By inspecting the destination IP addresses in the packet capture, you should be able to compile a list of IPs for each application. This list can then be used in the iptables rules.

Please note that this method might not give a comprehensive list of all IPs used by these apps, but it should give a reasonable subset for a proof-of-concept project.

Once you've obtained the IP ranges for each of these apps, the next step would be to modify the iptables rules in the redsocks setup script to include these IPs. Afterward, test the setup to ensure the apps' traffic is being correctly routed through the Nym socks5 proxy.

## <b>"TO-DO": 7.3 and on, traffic segmentation, wrapper for networking </b>