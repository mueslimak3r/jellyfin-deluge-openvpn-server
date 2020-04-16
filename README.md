# jellyfin-deluge-openvpn-server
Guide for setting up a media streaming server using deluge and openvpn for media downloading, jellyfin for streaming, and caddy for the proxy

When I set this up I was using a Raspberry Pi 4B (4GB) running the "dietpi" distro. This tutorial will probably work on many debian based systems. The primary non-root user is called "dietpi" in this tutorial.

This tutorial will guide you through the setup of openvpn, deluge, jellyfin, and caddy

prerequisites: screen, emacs/vim/nano, dropbear

**I highly recommend you have a heatsink for you Pi, and overclock it**

# Step 0, set up a static local IP for your raspberry pi

go to your router's admin page and configure the DHCP settings to reserve an IP address for your raspberry pi. It's easiest to check the current ip your Pi has and use that.



# Step 1, set up openvpn split tunnel

Follow the tutorial below to set up a split tunnel for openvpn. This works by creating a new user called "vpn" and routing all its services through openvpn. This is the easiest method I could find to run only deluge through a vpn while the other services (deluge web-ui, jellyfin, caddy, etc) are networked normally

https://www.htpcguides.com/force-torrent-traffic-vpn-split-tunnel-debian-8-ubuntu-16-04/



# Step 2, install deluge and make it run through vpn

while following the below tutorial, when creating "/etc/systemd/system/deluge-web.service", set the user and group variables to "dietpi" instead of "vpn".
```
User=dietpi

Group=dietpi
```


The reason for this change is to run the service through your primary user instead of through the tunneled user.

Since we will use caddy for serving this service and several others, don't do the section of the tutorial for setting up nginx. Stop after the prior section.

Follow the tutorial below, and at the end you should be able to connect to the web-ui but the daemon should show as offline. This is because **deluge stores auth files per user**, and since we have diverged from the tutorial and are running the web-ui as a different user, we need to make sure that our web-ui's user has a copy of the "vpn" user's auth file.

https://www.htpcguides.com/configure-deluge-for-vpn-split-tunneling-debian-8/

after completing that tutorial, copy the "vpn" user's auth file "/home/vpn/.config/deluge/auth" to "/home/dietpi/.config/deluge", replacing the existing one.

Once that is done restart both deluge services and make sure the web-ui works and can connect to the daemon. If that works go to the below site and follow the instructions to make sure deluge is properly tunneled and your IP is changed (compare the ip reported by the site to the ip you get when you google "what's my IP").

https://torguard.net/checkmytorrentipaddress.php



# Step 3, install jellyfin

follow the relevant steps in this tutorial. I skipped some because I'm using dietpi instead of raspbian (eg: running rpi-update won't work on dietpi).

https://www.electromaker.io/tutorial/blog/how-to-install-jellyfin-on-the-raspberry-pi

after this we need to make sure that jellyfin can access the files downloaded by deluge. Since deluge only gives read permissions to users in its group we'll need to add jellyfin's user "jellyfin" to the same group as the deluge daemon "vpn". To do this run:
```
usermod -a -G vpn jellyfin
```
restart the jellyfin service and set up your library. Now you'll need to make sure jellyfin's network settings are set up correctly for securely making it available over the internet.

Log into Jellyfin and go to "Dashboard" and find the "Networking" page. Disable "Allow remote connections to this Jellyfin Server". Caddy will be managing remote connection so as far as Jellyfin is concerned everything is local. Disabling this setting will still allow you to connect via your local network.

Now you'll need to set up hardware acceleration for video encoding/decoding. Some of these features aren't supported by raspberry pi models below 4. Follow this tutorial:

https://jellyfin.org/docs/general/administration/hardware-acceleration.html



**you can stop here if you are only using this on your lan**

# Step 4, open ports and set up DNS

In the next step you will set up caddy to proxy your services and serve them over the web through https (port 443). It also needs the http port open (80). Go to your router's admin panel and forward ports 443 and 80 (TCP/UDP) to your raspberry pi's local IP. **If you plan to remotely SSH into your pi make sure you disable root ssh login, change ssh login from password to RSA key, and change the port SSH uses from 22 to something less predictable.** You can generate one here:

https://www.random.org/integers/?num=1&min=5001&max=49151&col=5&base=10&format=html&rnd=new

Now configure your raspberry pi to use that port for SSH. I use dropbear so I set that in "/etc/default/dropbear", but for openssh set it in "/etc/ssh/sshd_config". Once you make sure that you can ssh using that port, forward that port through your router settings (TCP).

I used cloudflare's dns to set this up. In cloudflare, in the SSL/TLS tab I set it to "full (strict)". In the DNS tab I add two new 'A' records. "Name" is set to the name of the subdomain. This needs to match what you set in the Caddyfile when you install Caddy. In this case I have two subdomains. One for jellyfin and one for the deluge web-ui.
```
Type    Name      Content	                      TTL     Proxy

A       jellyfin  (public ip of raspberry pi)   Auto    Proxied

A       p2p       (public ip of raspberry pi)   Auto    Proxied
```


# Step 5, install caddy




Install Caddy:
```
sudo apt-get remove apache2
```
```
curl https://getcaddy.com | bash -s personal
```
```
sudo mkdir /etc/caddy
```
```
sudo touch /etc/caddy/My-CaddyFile
```
included in the repo is a copy of the caddyfile I used, with my personal info removed. Populate your caddy file. Make sure that for each subdomain the names match those set in your DNS (cloudflare)

now create a new session in screen. I'll call mine "caddy-server"
```
screen -S caddy-server
```
```
sudo /etc/caddy -conf /etc/caddy/My-CaddyFile
```
Make sure caddy is running properly and getting an SSL cert. Try connecting to your domain and make sure everything works.

Then finally you can detach the screen session with (ctrl + a, then press 'd'), and you can exit your terminal.
