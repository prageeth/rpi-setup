# Firewall
Make sure `iptables` is installed.

```
sudo iptables -L
```

## Add firewall rules
```
sudo nano /etc/iptables.firewall.rules
```

Save the following in the file:
```
*filter

#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT -d 127.0.0.0/8 -j REJECT

#  Accept all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#  Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 8080 -j ACCEPT

#  Allow SSH connections
#
#  The -dport number should be the same port number you set in sshd_config
#
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

#  Allow ping
-A INPUT -p icmp -j ACCEPT

#  Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

#  Drop all other inbound - default deny unless explicitly allowed policy
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT
```

### Load the firewall rules
```
sudo iptables-restore < /etc/iptables.firewall.rules
```
*NOTE* Verify by running `sudo iptables -L`

### Load everytime network adaptor is initialized
Make a new file in the network adaptor hooks by running
```
sudo nano /etc/network/if-pre-up.d/firewall
```
Add the following:
```
#!/bin/sh

/sbin/iptables-restore < /etc/iptables.firewall.rules
```
*NOTE* Make sure the file is executable:
```
sudo chmod +x /etc/network/if-pre-up.d/firewall
```

## Defend Against Brute Force Attacks
We will use `Fail2Ban` utility to block malacious users trying a brute force entry using SSH.

```
sudo apt-get install fail2ban -y
```