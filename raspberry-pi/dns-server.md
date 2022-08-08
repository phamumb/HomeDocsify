## Introduction

A Domain Name System (DNS) translates human-readable domain names to IP addresses. DNS serves as a phone book of internet addresses for quicker access to pages. Setting up a Raspberry Pi as a DNS server improves DNS lookup time and connection speed.

This guide explains how to configure the Raspberry Pi as a DNS server.
## Prerequisites
- Raspberry Pi 2, 3, or 4 with Raspbian OS using a static IP address
- Ethernet cable connection or Wi-Fi dongle
- Power supply
- MicroSD card
- Terminal access (directly or through SSH) with sudo privileges

## Raspberry Pi DNS Server Setup Guide
### Step 1: Update Packages
Before starting, open the terminal and update software packages on your Raspberry Pi using the apt package manager:
```console 
sudo apt update
sudo apt upgrade
```
### Step 2: Install DNS Software
Install DNSMasq on the Raspberry Pi:
```console 
sudo apt install dnsmasq
```
DNSMasq is an excellent choice for small-scale networks.

### Step 3: Configure DNSMasq

Configuring DNSMasq ensures the best setup for the DNS server.

1. Modify the dnsmasq.conf file using the nano text editor by running:
```console 
sudo nano /etc/dnsmasq.conf
```
2. Locate (CTRL+W to search) and uncomment the following lines by removing the hash sign (#):
    - `domain-needed` – Configures the DNS server to not forward names without a dot (.) or a domain name to upstream servers. Any names without a dot or domain stay in the local network.
    - `bogus-priv` – Stop DNS server from forwarding local IP range reverse-lookup queries to upstream DNS servers. That prevents leaking of the local network to upstream servers.
    - `no-resolv` – Stops reading the upstream nameservers from the /etc/resolv.conf file, relying instead on the ones in the DNSMasq configuration.
3. Replacing DNS Server 
```console
server=8.8.8.8
server=8.8.4.4
```
4. Find following line and replace with `1000`:
```console
cache-size=1000
```
5. Save the file with CTRL+X, then press Y and hit Enter to keep the changes.

6. Restart DNSMasq to apply the changes:
```console
sudo systemctl restart dnsmasq
```
7. Check DNS status:
```console
sudo systemctl status dnsmasq
```
The status shows as `active (running)`, indicating the Raspberry Pi is running as a DNS server: