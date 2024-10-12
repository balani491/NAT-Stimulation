
# Network Topology Setup Using Linux Network Namespaces

## Overview

This document provides a step-by-step guide for setting up a network topology using Linux network namespaces and iptables to implement NAT, port forwarding, and traffic restriction. It includes instructions for creating namespaces, configuring network interfaces, and running a simple HTTP server for testing.

## Prerequisites

- Linux operating system with `iproute2` and `iptables` installed.
- Python 3 for running a simple HTTP server.

## Setup Instructions

### 1. Create Namespaces

```bash
sudo ip netns add client
sudo ip netns add router
```

### 2. Create veth Pairs

```bash
sudo ip link add veth-client type veth peer name veth-router
```

### 3. Move Interfaces to Their Respective Namespaces

```bash
sudo ip link set veth-client netns client
sudo ip link set veth-router netns router
```

### 4. Configure the Client Interface

```bash
sudo ip netns exec client ip addr add 192.168.10.2/24 dev veth-client
sudo ip netns exec client ip link set veth-client up
```

### 5. Configure the Router Interface

```bash
sudo ip netns exec router ip addr add 192.168.10.1/24 dev veth-router
sudo ip netns exec router ip link set veth-router up
```

### 6. Create a Loopback Interface for the Router

```bash
sudo ip netns exec router ip addr add 10.0.0.1/24 dev lo
sudo ip netns exec router ip link set lo up
```

### 7. Enable IP Forwarding in the Router Namespace

```bash
sudo ip netns exec router sysctl -w net.ipv4.ip_forward=1
```

### 8. Configure NAT on the Router

```bash
# Assuming the external interface is eth0
sudo ip netns exec router iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### 9. Allow HTTP and HTTPS Traffic

```bash
sudo ip netns exec router iptables -A FORWARD -p tcp -m tcp --dport 80 -j ACCEPT
sudo ip netns exec router iptables -A FORWARD -p tcp -m tcp --dport 443 -j ACCEPT
```

### 10. Run a Simple HTTP Server on the Client

```bash
sudo ip netns exec client python3 -m http.server 80
```

## Testing the Setup

1. From the router namespace, you can ping the client to ensure connectivity:
   ```bash
   sudo ip netns exec router ping 192.168.10.2
   ```
2. Access the HTTP server from an external machine using the router's external IP.

## Issues Encountered

- **Network Interface Issues**: Sometimes, the interfaces may not be properly configured if the commands are not run in the correct order or with the appropriate permissions.
- **Firewall Rules**: Incorrect iptables rules can prevent traffic from being forwarded properly, leading to connectivity issues.
- **Python HTTP Server**: Ensure that Python is installed and accessible in the client namespace for the HTTP server to work.
- **Namespace Deletion**: If the namespaces need to be deleted, ensure that all interfaces are properly removed before attempting to delete them.



![p1](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143001.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143008.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143015.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143024.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143031.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143037.png)
![](https://github.com/balani491/NAT-Stimulation/blob/main/Screenshot%202024-10-12%20143042.png)
