# "Always on" OpenVPN on Linux

## Install necessary packages

```console
apt-get install dnsmasq iptables-persistent ntp openresolv openvpn
```

## NTP

Accurate time is important for the VPN encryption to work. If the VPN client's clock is too far off, the VPN server will reject the client.

You shouldn't have to do anything to set this up, when the `ntp` service is installed, it is enabled by default.

Double-check your machine is getting the correct time from internet time servers with `ntpq -p`, you should see at least one peer with a `+` or a `*` or an `o`, for example:

```console
$ ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
-0.time.xxxx.com 104.21.137.30    2 u   47   64    3  240.416    0.366   0.239
+node01.jp.xxxxx 226.252.532.9    2 u   39   64    7  241.030   -3.071   0.852
*t.time.xxxx.net 104.1.306.769    2 u   38   64    7  127.126   -2.728   0.514
+node02.jp.xxxxx 250.9.592.830    2 u    8   64   17  241.212   -4.784   1.398
```

## OpenVPN

### Setup VPN Client

Download a `*.ovpn` profile (and certificates if they are in separate files) for your provider:

```console
wget https://provider.vpn.com/profile.ovpn
```

Copy the OpenVPN profile (and optional certificates) to the OpenVPN client:

```console
sudo cp certificate.crt provider.ca /etc/openvpn # If necessary
sudo cp profile.ovpn /etc/openvpn/profile.conf
```

You can use a diffrent VPN endpoint if you like (instead of `profile`). Note the extension change from **ovpn** to **conf**.

Create `/etc/openvpn/login` containing only your username and password, one per line, for example:

```text
user12345678
MyGreatPassword
```

Change the permissions on this file so only the root user can read it:

```console
sudo chmod 600 /etc/openvpn/login
```

Setup OpenVPN to use your stored username and password by editing the the config file for the VPN endpoint:

```console
sudo vim /etc/openvpn/profile.conf
```

Change the `auth-user-pass` line to `auth-user-pass /etc/openvpn/login`.

### Test VPN

At this point you should be able to test the VPN actually works:

```console
sudo openvpn --config /etc/openvpn/profile.conf
```

If all is well, you'll see something like:

```console
$ sudo openvpn --config /etc/openvpn/profile.conf
Sat Oct 24 12:10:54 2015 OpenVPN 2.3.4 arm-unknown-linux-gnueabihf [SSL (OpenSSL)] [LZO] [EPOLL] [PKCS11] [MH] [IPv6] built on Dec  5 2014
Sat Oct 24 12:10:54 2015 library versions: OpenSSL 1.0.1k 8 Jan 2015, LZO 2.08
Sat Oct 24 12:10:54 2015 UDPv4 link local: [undef]
Sat Oct 24 12:10:54 2015 UDPv4 link remote: [AF_INET]123.123.123.123:1194
Sat Oct 24 12:10:54 2015 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Sat Oct 24 12:10:56 2015 [Private Internet Access] Peer Connection Initiated with [AF_INET]123.123.123.123:1194
Sat Oct 24 12:10:58 2015 TUN/TAP device tun0 opened
Sat Oct 24 12:10:58 2015 do_ifconfig, tt->ipv6=0, tt->did_ifconfig_ipv6_setup=0
Sat Oct 24 12:10:58 2015 /sbin/ip link set dev tun0 up mtu 1500
Sat Oct 24 12:10:58 2015 /sbin/ip addr add dev tun0 local 10.10.10.6 peer 10.10.10.5
Sat Oct 24 12:10:59 2015 Initialization Sequence Completed
```

Exit this with **Ctrl+c**

### Enable VPN at boot

```console
sudo systemctl enable openvpn@profile
```

## DNS

Add DNS configuration to `/etc/openvpn/profile.conf`:

```ini
# First three lines are optional, and they depend if your VPN provider provides those values or not.
# If not, use DNS servers provided by your provider to avoid DNS leaks!
dhcp-option DNS 208.67.222.222
dhcp-option DNS 208.67.220.220
dhcp-option DOMAIN-ROUTE .

script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
down-pre
```

Configure dnsmasq (use DNS servers provided by your OpenVPN provider):

```console
echo 'no-resolv' >> /etc/dnsmasq.conf
echo 'no-poll' >> /etc/dnsmasq.conf
echo 'server=208.67.222.222' >> /etc/dnsmasq.conf
echo 'server=208.67.220.220' >> /etc/dnsmasq.conf
```

## Routing and NAT

Enable IP Forwarding:

```console
echo -e '\n#Enable IP Routing\nnet.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Setup NAT from the local LAN down the VPN tunnel:

```console
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo iptables -A FORWARD -i tun0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT
```

Make NAT rules persistent across reboot:

```console
sudo netfilter-persistent save
```

Make the rules apply at startup:

```console
sudo systemctl enable netfilter-persistent
```

### VPN Kill Switch

This will block outbound traffic from your machine so that only the VPN and related services are allowed.

Once this is done, the only way your machine can get to the internet is over the VPN.

This means if the VPN goes down, your traffic will just stop working, rather than end up routing over your regular internet connection where it could become visible.

```console
sudo iptables -A OUTPUT -o tun0 -m comment --comment "vpn" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p icmp -m comment --comment "icmp" -j ACCEPT
sudo iptables -A OUTPUT -d 192.168.1.0/24 -o eth0 -m comment --comment "lan" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 1194 -m comment --comment "openvpn" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp -m tcp --sport 22 -m comment --comment "ssh" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 123 -m comment --comment "ntp" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 53 -m comment --comment "dns" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp -m tcp --dport 53 -m comment --comment "dns" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -j DROP
```

And save so they apply at reboot:

```console
sudo netfilter-persistent save
```

After all steps, reboot your machine and you should be good to go!

## Troubleshooting

### OpenVPN problems

If you find traffic on your other systems stops, then look on your machine to see if the VPN is up or not.

You can check the status and logs of the VPN client with:

```console
sudo systemctl status openvpn@profile
sudo journalctl -u openvpn@profile
```

### Avoiding DNS leaks

First of all, check if `openresolv` is working correctly:

```console
sudo cat /etc/resolv.conf
```

You should see only one nameserver: `nameserver 127.0.0.1`.

Next check check `dnsmasq` logs:

```console
sudo cat /var/log/syslog | grep dnsmasq
```

You should see `using nameserver xxx.xxx.xxx.xxx#53` along the lines, and only VPN DNS servers should be listed.
If not, ensure that `no-resolv` and `no-poll` directives are enabled in `dnsmasq.conf`.

Sometimes when you do [DNS Leak test](https://www.dnsleaktest.com/), browser may hit it's internal DNS cache.
Ensure that's it's flushed out before doing the testing.

