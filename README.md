<p align="left">
  <img src="wireguard-pihole-banner.png" alt="Banner" width="1000" />
</p>
<br>
How to setup WireGuard with Pi-hole and Netdata for statistics (and n8n as a bonus :))

This runs completely free of charge on an Ubuntu 24.04 instance on Oracle Cloud Free Tier (shape VM.Standard.A1.Flex: 1 OCPU ; 6 GB memory ; 10 TB p/m free outbound internet traffic)

The webpages for Pi-hole, Netdata and n8n are only reachable on connected VPN clients!
<br>
<br>
<br>
<h2>Update Ubuntu first!</h2>
sudo apt update && sudo apt upgrade -y
<br>
<br>
<h2>Step 1: Add Ubuntu Firewall Rule for WireGuard internet access</h2>
  <i>NOTE: open this port in Oracle Cloud security list too!</i><br>
  <br>
sudo iptables -I INPUT 2 -p udp --dport 51820 -j ACCEPT<br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
<br>
<br>
<h2>Step 2: Install Pi-hole</h2>
<i>Install Pi-hole first so PiVPN can pick it up automatically while installing in step 3!</i><br>
<br>
sudo curl -sSL https://install.pi-hole.net | bash<br>
Follow steps in wizard<br>
<br>
Make Pi-hole admin page reachable for connected VPN clients:<br>
sudo iptables -I INPUT 3 -i wg0 -p tcp --dport 80 -j ACCEPT <i>(NOTE: must be above REJECT rule in INPUT section!->check with: sudo iptables -L -v -n)</i><br>
sudo iptables -I INPUT 4 -i wg0 -p tcp --dport 443 -j ACCEPT <i>(NOTE: must be above REJECT rule in INPUT section!->check with: sudo iptables -L -v -n)</i><br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
<br>
<u>Pi-hole extra blocklists:</u><br>
from https://blog.burstbytes.com.au/best-2026-pi-hole-blocklists-and-how-to-install-them/#How_to_install_a_Pi-hole_blocklist:<br>
- https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt<br>
- https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/popupads.txt<br>
- https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt<br>
- https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/fake.txt<br>
<br>
from https://github.com/zachlagden/Pi-hole-Optimized-Blocklists:<br>
- https://media.githubusercontent.com/media/zachlagden/Pi-hole-Optimized-Blocklists/main/lists/all_domains.txt<br>
<br>
from ex-colleague Paulus1337 (thanks Paul! ;-)):<br>
- https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts<br>
- http://sysctl.org/cameleon/hosts<br>
- https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt<br>
- https://justdomains.github.io/blocklists/lists/easylist-justdomains.txt<br>
- https://justdomains.github.io/blocklists/lists/easyprivacy-justdomains.txt<br>
- https://justdomains.github.io/blocklists/lists/adguarddns-justdomains.txt<br>
- https://justdomains.github.io/blocklists/lists/nocoin-justdomains.txt<br>
- https://nsfw.oisd.nl<br>
- https://big.oisd.nl<br>
- https://raw.githubusercontent.com/dashingdon/blocklists/refs/heads/master/pihole_addonblacklist.txt<br>
<br>
<br>
<h2>Step 3: Install WireGuard with PiVPN</h2>
  <i>or WG-Easy (=WireGuard + management GUI with Docker (not covered here))</i><br>
<br>
sudo curl -L https://install.pivpn.io | bash<br>
Follow steps in wizard (choose WireGaurd instead of OpenVPN!)<br>
Now run 'pivpn add' to create the client profiles<br>
<br>
<u>Profile for iPhone example:</u><br>
pivpn add iphone-vpn<br>
pivpn -qr iphone-vpn<br>
Scan QR code with the WireGuard app on your iPhone<br>
<br>
<u>Profile for laptop example:</u><br>
pivpn add laptop-vpn<br>
cat ~/configs/laptop-vpn.conf<br>
Highlight and copy the entire output text from the terminal (starting from [Interface] down to the last line)<br>
On your Windows laptop, open Notepad (or any text editor)<br>
Paste the text into Notepad.<br>
Save the file as wg0.conf<br>
Import this file into the WireGuard windows application
<br>
<br>
<h2>Step 4: Install Netdata (Real-Time Server Performance Monitoring)</h2>
<i>Netdata provides an incredible, low-overhead real-time monitoring dashboard that will give you instant visibility into your network traffic</i><br>
<br>
sudo wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh<br>
sudo iptables -I INPUT 5 -i wg0 -p tcp --dport 19999 -j ACCEPT <i>(NOTE: must be above REJECT rule in INPUT section!->check with: sudo iptables -L -v -n)</i><br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
<br>
<br>
<h2>Step 5: Install n8n (Workflow Automation)</h2>
<i>n8n is a visual, node-based workflow automation platform that connects different apps, APIs, and AI agents so they can exchange data and run tasks automatically.</i><br>
<br>
mkdir -p ~/n8n<br>
cd ~/n8n<br>
nano docker-compose.yml<br>
Paste the code and save <i>(NOTE:put N8N_SECURE_COOKIE=false in yml to be able to access site over http)</i><br>
sudo apt install -y docker.io docker-compose-v2 (Install Docker and Docker Compose plugin)<br>
sudo systemctl enable --now docker (Start and enable Docker service)<br>
sudo usermod -aG docker $USER (Allow the ubuntu user to run Docker without sudo)<br>
newgrp docker (Allow the ubuntu user to run Docker without sudo)<br>
docker compose up -d (Start the service using Docker Compose)<br>
docker ps (Verify that n8n is running properly)<br>
sudo iptables -I INPUT 6 -i wg0 -p tcp --dport 5678 -j ACCEPT <i>(NOTE: must be above REJECT rule in INPUT section!->check with: sudo iptables -L -v -n)</i><br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
<br>
<i>docker compose down && docker compose up -d (if needed: This will gracefully shut down the container and spin it right back up with a clean restart)</i><br>
<br>
<br>
<h2>The results</h2>
<br>
<i>Pi-hole</i>
<br>
<br>
<p align="left">
  <img src="dashboard-pihole.png" alt="Pi-hole Dashboard" width="600" />
</p>
<br>
<i>Netdata</i>
<br>
<br>
<p align="left">
  <img src="dashboard-netdata.png" alt="Netdata Dashboard" width="600" />
</p>
<br>
<i>n8n</i>
<br>
<br>
<p align="left">
  <img src="dashboard-n8n.png" alt="n8n Dashboard" width="600" />
</p>

