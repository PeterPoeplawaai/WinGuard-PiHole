How to setup WireGuard with PiHole and Netdata for statistics (and n8n as a bonus :))

This runs completely free of charge on an Ubuntu instance on Oracle Cloud Free Tier: 1 OCPU ; 6 GB memory ; 10 TB p/m free outbound internet traffic.

The webpages for PiHole, Netdata and n8n are only reachable on connected VPN clients!




<h1>Update Ubuntu first!</h1>
sudo apt update && sudo apt upgrade -y


<h1>Step 1: Add Internal Ubuntu Firewall Rule (NOTE: open this port in Oracle Cloud security list too!)</h1>
sudo iptables -I INPUT 2 -p udp --dport 51820 -j ACCEPT
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)


<h1>Step 2: Install Pi-Hole</h1>
sudo curl -sSL https://install.pi-hole.net | bash
Make Pi-Hole admin page reachable for connected VPN clients:
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 80 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 443 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)

<u>Pi-Hole handy CLI commands:</u>
sudo pihole enable
sudo pihole -g (of pihole -up) = update pi-hole and blacklists
sudo pihole -t = live logs

<u>Pi-Hole extra blocklists:</u>
from https://blog.burstbytes.com.au/best-2026-pi-hole-blocklists-and-how-to-install-them/#How_to_install_a_Pi-hole_blocklist:
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/popupads.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/fake.txt

from https://github.com/zachlagden/Pi-hole-Optimized-Blocklists:
https://media.githubusercontent.com/media/zachlagden/Pi-hole-Optimized-Blocklists/main/lists/all_domains.txt

from ex-colleague Paulus1337 (thanks Paul! ;-)):
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
http://sysctl.org/cameleon/hosts
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://justdomains.github.io/blocklists/lists/easylist-justdomains.txt
https://justdomains.github.io/blocklists/lists/easyprivacy-justdomains.txt
https://justdomains.github.io/blocklists/lists/adguarddns-justdomains.txt
https://justdomains.github.io/blocklists/lists/nocoin-justdomains.txt
https://nsfw.oisd.nl
https://big.oisd.nl
https://raw.githubusercontent.com/dashingdon/blocklists/refs/heads/master/pihole_addonblacklist.txt


<h1>Step 3: Install WireGuard with PiVPN (Debian/Ubuntu only!!)</h1>
  <h2>(or WG-Easy (=WireGuard + management GUI met Docker))</h2>
sudo curl -L https://install.pivpn.io | bash
Follow steps in wizard (choose WireGaurd instead of OpenVPN!)
Now run 'pivpn add' to create the client profiles
Run 'pivpn help' to see what else you can do!

<u>Profile for iPhone</u>
pivpn add
pivpn -qr iphone-vpn

<u>Profile for laptop</u>
pivpn add
cat ~/configs/laptop-vpn.conf
Highlight and copy the entire output text from the terminal (starting from [Interface] down to the last line).
On your Windows laptop, open Notepad (or any text editor).
Paste the text into Notepad.
Save the file as wg0.conf


<h1>Step 4: Install Netdata (Real-Time Server Performance Monitoring)</h1>
## Netdata provides an incredible, low-overhead real-time monitoring dashboard that will give you instant visibility into your network traffic
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 19999 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)


<h1>Step 5: Install n8n (Workflow Automation)</h1>
mkdir -p ~/n8n
cd ~/n8n
nano docker-compose.yml
Paste the code and save (NOTE: put N8N_SECURE_COOKIE=false in yml to be able to access site over http)
sudo apt install -y docker.io docker-compose-v2 (Install Docker and Docker Compose plugin)
sudo systemctl enable --now docker (Start and enable Docker service)
sudo usermod -aG docker $USER (Allow the ubuntu user to run Docker without sudo)
newgrp docker (Allow the ubuntu user to run Docker without sudo)
docker compose up -d (Start the service using Docker Compose)
docker ps (Verify that n8n is running properly)
sudo iptables -I INPUT 8 -i wg0 -p tcp --dport 5678 -j ACCEPT (NOTE: must be above REJECT rule in INPUT section! -> check with: sudo iptables -L -v -n)
sudo netfilter-persistent save (Make your iptables rules persistent across reboots)
## docker compose down && docker compose up -d (if needed: This will gracefully shut down the container and spin it right back up with a clean restart)

