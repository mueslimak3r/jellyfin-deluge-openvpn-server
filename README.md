# jellyfin-deluge-openvpn-server
Guide for setting up a media streaming server using deluge and openvpn for media downloading, jellyfin for streaming, and caddy for the proxy

when creating "/etc/systemd/system/deluge-web.service" set

User=dietpi
Group=dietpi

You can replace "dietpi" with whatever user, other than "vpn" (so it isn't tunneled). The user you choose should be non-root.

The reason for this change is to run the service throught your primary user, not through the tunneled user.

Since we will use caddy for serving this service and several others, don't do the section of the tutorial for setting up nginx. Stop after the prior section.

Follow the tutorial below, and at the end you should be able to connect to the web-ui but the daemon should show as offline. This is because deluge stores auth files per user, and since we have diverged from the tutorial and are running the web-ui as a different user, we need to make sure that our web-ui's user has a copy of the "vpn" user's auth file.

https://www.htpcguides.com/configure-deluge-for-vpn-split-tunneling-debian-8/

