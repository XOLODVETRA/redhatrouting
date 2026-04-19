#Configuring NAT and Routing in Linux using nmtui, sysctl, and nftables

This guide explains how to configure a Linux server to act as a router (gateway) that shares internet access with other devices. Standard tools are used: nmtui, sysctl, and nftables.

1. Configuring Network Interfaces via nmtui

Command:

nmtui

This is a text-based interface for network management (part of NetworkManager).

What is done here:
Configure network adapters
Assign IP addresses
Rename interfaces (e.g., ens160, ens192)
Specify which interface will be:
external (WAN) — with internet access
internal (LAN) — for the local network

👉 Example:

ens160 — internet
ens192 — local network
2. Applying Settings
Why this is needed:

Restarts the current shell (bash) to:

apply new environment variables
refresh network settings

(Not always required, but often used after network changes)

3. Enabling IP Forwarding

Open the file:

nano /etc/sysctl.conf

Add or modify the line:

net.ipv4.ip_forward = 1
What it means:

This enables packet routing between interfaces.

📌 Without this:

the computer will NOT forward packets
it behaves like a regular host

📌 With this:

it becomes a router

Apply the settings:

sysctl -p
4. Configuring NAT with nftables

Create a file, for example:

/etc/nftables/my_nat.nft

Add:

table inet nat {
    chain POSTROUTING {
        type nat hook postrouting priority srcnat;
        oifname "ens160" masquerade
    }
}
Configuration Breakdown

table inet nat
Creates a NAT table (for IPv4 and IPv6)

chain POSTROUTING
A chain that processes packets after routing

type nat hook postrouting
Specifies this as a NAT chain
postrouting — applied before the packet leaves

oifname "ens160"
Condition: applies only to packets going out through interface ens160

masquerade
Enables masquerading (NAT)

👉 This means:

All devices in the local network will access the internet using the server’s IP address.

5. Enabling nftables Configuration

Edit the file:

nano /etc/sysconfig/nftables.conf

Find the line:

# include "/etc/nftables/main.nft"
What to do:
Remove #
Replace with your file:
include "/etc/nftables/my_nat.nft"
6. Starting and Enabling nftables
systemctl enable --now nftables
What this command does:
enable — enables autostart at boot
--now — starts the service immediately
7. Result

After completing all steps, the system:

✅ Forwards packets (IP forwarding)
✅ Performs NAT (masquerading)
✅ Functions as a full router

8. How It Works
A device in the LAN sends a packet
The server receives it
Forwards it through WAN (ens160)
Rewrites the IP (masquerade)
The internet responds to the server
The server sends the response back to the client
9. Typical Setup
[ LAN Clients ]
        ↓
   ens192 (LAN)
   [ Linux Server ]
   ens160 (WAN)
        ↓
     Internet
10. Common Mistakes

❌ IP forwarding not enabled
❌ Wrong interface specified (ens160)
❌ nftables not running
❌ Configuration not included in nftables.conf

11. Verification

Check NAT rules:

nft list ruleset

Check IP forwarding:

cat /proc/sys/net/ipv4/ip_forward

Expected output:

1
Conclusion

You have configured:

Network interfaces (nmtui)
Routing (sysctl)
NAT (nftables)
Autostart (systemctl)

This is a classic Linux gateway/router setup — the foundation for servers, virtualization, and lab networks.
