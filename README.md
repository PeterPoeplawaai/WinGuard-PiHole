How to setup WireGuard with PiHole and Netdata for statistics (and n8n as a bonus :))

This runs completely free of charge on an Ubuntu instance on Oracle Cloud Free Tier: 1 OCPU ; 6 GB memory ; 10 TB p/m free outbound internet traffic.

The webpages for PiHole, Netdata and n8n are only reachable on connected VPN clients!




<h1>Update Ubuntu first!</h1>
sudo apt update && sudo apt upgrade -y


<h1>Step 1: Add Internal Ubuntu Firewall Rule (NOTE: open this port in Oracle Cloud security list too!)</h1>
sudo iptables -I INPUT 2 -p udp --dport 51820 -j ACCEPT<br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
<br>
<br>
<h1>Step 2: Install Pi-Hole</h1>
sudo curl -sSL https://install.pi-hole.net | bash<br>
Make Pi-Hole admin page reachable for connected VPN clients:<br>
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 80 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)<br>
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 443 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)<br>
<br>
<u>Pi-Hole handy CLI commands:</u><br>
- sudo pihole enable<br>
- sudo pihole -g (of pihole -up) = update pi-hole and blacklists<br>
- sudo pihole -t = live logs<br>
<br>
<u>Pi-Hole extra blocklists:</u><br>
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
<h1>Step 3: Install WireGuard with PiVPN (Debian/Ubuntu only!!)</h1>
  <h3>or WG-Easy (=WireGuard + management GUI with Docker</h3>
sudo curl -L https://install.pivpn.io | bash<br>
Follow steps in wizard (choose WireGaurd instead of OpenVPN!)<br>
Now run 'pivpn add' to create the client profiles<br>
Run 'pivpn help' to see what else you can do!<br>
<br>
<u>Profile for iPhone</u><br>
pivpn add<br>
pivpn -qr iphone-vpn<br>
<br>
<u>Profile for laptop</u><br>
pivpn add<br>
cat ~/configs/laptop-vpn.conf<br>
Highlight and copy the entire output text from the terminal (starting from [Interface] down to the last line).<br>
On your Windows laptop, open Notepad (or any text editor).<br>
Paste the text into Notepad.<br>
Save the file as wg0.conf<br>
<br>
<br>
<h1>Step 4: Install Netdata (Real-Time Server Performance Monitoring)</h1>
## Netdata provides an incredible, low-overhead real-time monitoring dashboard that will give you instant visibility into your network traffic<br>
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 19999 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)<br>
<br>
<br>
<h1>Step 5: Install n8n (Workflow Automation)</h1>
mkdir -p ~/n8n<br>
cd ~/n8n<br>
nano docker-compose.yml<br>
Paste the code and save (NOTE: put N8N_SECURE_COOKIE=false in yml to be able to access site over http)<br>
sudo apt install -y docker.io docker-compose-v2 (Install Docker and Docker Compose plugin)<br>
sudo systemctl enable --now docker (Start and enable Docker service)<br>
sudo usermod -aG docker $USER (Allow the ubuntu user to run Docker without sudo)<br>
newgrp docker (Allow the ubuntu user to run Docker without sudo)<br>
docker compose up -d (Start the service using Docker Compose)<br>
docker ps (Verify that n8n is running properly)<br>
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 5678 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)<br>
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)<br>
## docker compose down && docker compose up -d (if needed: This will gracefully shut down the container and spin it right back up with a clean restart)<br>

