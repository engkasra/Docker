# Docker Swarm Hostname Configuration Guide

This guide explains how to replace default hostnames (like `debian`) with meaningful names or IP addresses in `docker node ls` output.

## Problem

By default, Docker Swarm shows system hostnames in the HOSTNAME column:
```bash
$ docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS
abc123...                     debian     Ready     Active         Leader
def456...                     debian     Ready     Active         Reachable
```

## Solution Overview

The HOSTNAME column displays the system's hostname from the OS level. To change it:

1. Update system hostname on each node
2. Update `/etc/hosts` for DNS resolution across the cluster
3. Restart Docker service

## Method 1: Using IP Addresses

### Extract Node IP

**❌ Unreliable method:**
bash
NODE_IP=$(hostname -I | awk '{print $1}')  # May return Docker bridge IPs

**✅ Reliable method:**
bash
# Replace ens160 with your network interface name
NODE_IP=$(ip -4 addr show ens160 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

Or use routing table:
bash
NODE_IP=$(ip route get 8.8.8.8 | grep -oP 'src \K[\d.]+')

### Set Hostname

```bash
sudo hostnamectl set-hostname $NODE_IP
sudo systemctl restart docker
```
### Potential Issue: Terminal Display

After setting hostname to IP, you might see garbled output in terminal prompt:

kasra@10227161162:~$  # Missing dots

**Fix:** Ensure UTF-8 locale is configured:
```bash
locale  # Check current locale
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8
```
Also check terminal font supports all characters.

## Method 2: Using Descriptive Names (Recommended)

Instead of raw IPs, use meaningful names:

- Managers: `swarm-manager-1`, `swarm-manager-2`
- Workers: `swarm-worker-1`, `swarm-worker-2`
- Or with IP suffix: `manager-10-227-161-162`

### Automated Script

Create `setup-swarm-hostnames.sh`:

```bash
#!/bin/bash

# Node configuration map
declare -A NODES=(
["10.227.161.162"]="Swarm-Manager-161-162"
["10.227.161.163"]="Swarm-Worker-161-163"
["10.227.161.164"]="Swarm-Worker-161-164"
["10.227.161.165"]="Swarm-Worker-161-165"
)

# Detect current node IP
INTERFACE="ens160"  # Change to your interface name
CURRENT_IP=$(ip -4 addr show $INTERFACE | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

if [ -z "$CURRENT_IP" ]; then
echo "Error: Could not detect IP address on interface $INTERFACE"
exit 1
fi

# Get hostname for this node
NEW_HOSTNAME="${NODES[$CURRENT_IP]}"

if [ -z "$NEW_HOSTNAME" ]; then
echo "Error: IP $CURRENT_IP not found in NODES configuration"
exit 1
fi

echo "Setting hostname to: $NEW_HOSTNAME"
sudo hostnamectl set-hostname "$NEW_HOSTNAME"

# Backup /etc/hosts
sudo cp /etc/hosts /etc/hosts.backup.$(date +%Y%m%d_%H%M%S)

# Update /etc/hosts
echo "Updating /etc/hosts..."
sudo sed -i '/# Swarm Cluster Nodes/,/# End Swarm Cluster/d' /etc/hosts

{
echo ""
echo "# Swarm Cluster Nodes"
for ip in "${!NODES[@]}"; do
echo "$ip    ${NODES[$ip]}"
done
echo "# End Swarm Cluster"
} | sudo tee -a /etc/hosts > /dev/null

# Restart Docker
echo "Restarting Docker service..."
sudo systemctl restart docker

echo "Done! Hostname set to $NEW_HOSTNAME"
echo "Verify with: docker node ls"
```
### Deploy Script to All Nodes

```bash
# Copy script to all nodes
for ip in 10.227.161.162 10.227.161.163 10.227.161.164 10.227.161.165; do
scp setup-swarm-hostnames.sh kasra@$ip:/tmp/
done
```
# Execute on each node
```bash
for ip in 10.227.161.162 10.227.161.163 10.227.161.164 10.227.161.165; do
ssh kasra@$ip "chmod +x /tmp/setup-swarm-hostnames.sh && /tmp/setup-swarm-hostnames.sh"
done
```
## Verification

Check updated hostnames:
```bash
docker node ls

Expected output:

ID                            HOSTNAME                    STATUS    AVAILABILITY   MANAGER STATUS
abc123...                     Swarm-Manager-161-162       Ready     Active         Leader
def456...                     Swarm-Worker-161-163        Ready     Active         
ghi789...
```
