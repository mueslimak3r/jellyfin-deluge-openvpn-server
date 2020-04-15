# jellyfin-deluge-openvpn-server
Guide for setting up a media streaming server using deluge and openvpn for media downloading, jellyfin for streaming, and caddy for the proxy

# Step 1, set up openvp split tunnel

Follow the tutorial below to set up a split tunnel for openvpn. This works by creating a new user called "vpn" and routing all its services through openvpn. This is the easier method I could find to run only deluge through a vpn while the other services (deluge web-ui, jellyfin, caddy, etc) run and are networked normally

https://www.htpcguides.com/force-torrent-traffic-vpn-split-tunnel-debian-8-ubuntu-16-04/


# Step 2, install deluge and make it run through vpn

while follow the below tutorial, when creating "/etc/systemd/system/deluge-web.service" set

User=dietpi

Group=dietpi

You can replace "dietpi" with whatever user, other than "vpn" (so it isn't tunneled). The user you choose should be non-root.

The reason for this change is to run the service throught your primary user, not through the tunneled user.

Since we will use caddy for serving this service and several others, don't do the section of the tutorial for setting up nginx. Stop after the prior section.

Follow the tutorial below, and at the end you should be able to connect to the web-ui but the daemon should show as offline. This is because deluge stores auth files per user, and since we have diverged from the tutorial and are running the web-ui as a different user, we need to make sure that our web-ui's user has a copy of the "vpn" user's auth file.

https://www.htpcguides.com/configure-deluge-for-vpn-split-tunneling-debian-8/

after that copy the "vpn" user's auth file "/home/vpn/.config/deluge/auth" to "/home/dietpi/.config/deluge", replacing the existing one.

Once that is done restart both deluge services and make sure the web-ui works and can connect to the daemon. If that works go to the below site and follow the instructions to make sure deluge is properly tunneled and your IP is changed (compare the ip reported by the site to the ip you get when you google "what's my IP").

# Step 3, install jellyfin


