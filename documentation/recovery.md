# EBS Offline Recovery Procedure

## Scenario

The EC2 instance became inaccessible through SSH because the Ubuntu host firewall was allowing SSH only from an outdated public IP address.

Because normal remote access was unavailable, the root EBS volume was repaired from a temporary recovery instance.

## Recovery Architecture

Original EC2
     │
     │ Root EBS Volume
     ▼
Temporary Recovery EC2
     │
     ├── Mount original filesystem
     ├── Inspect configuration
     ├── Repair UFW rule
     └── Unmount filesystem
     │
     ▼
Original EBS Volume
     │
     ▼
Original EC2
     │
     ▼
SSH Restored

## Procedure

# 1. Stop the affected instance

The original EC2 instance was stopped before detaching its root volume.

# 2. Identify the EBS volume

The affected root volume was:

vol-0adc2e47489fc76c6

# 3. Launch a recovery instance

A temporary Ubuntu EC2 instance was launched in the same Availability Zone.

# 4. Attach the EBS volume

The original volume was attached to the recovery instance.

The operating system identified:

/dev/nvme1n1

as the original server disk.

The root partition was:

/dev/nvme1n1p1

# 5. Mount the filesystem

```
sudo mkdir /mnt/oldserver
sudo mount /dev/nvme1n1p1 /mnt/oldserver
```

The original filesystem became available at:

/mnt/oldserver

# 6. Inspect SSH configuration

```
sudo grep -RniE '^(Port|ListenAddress|PasswordAuthentication|PubkeyAuthentication|PermitRootLogin)' \
/mnt/oldserver/etc/ssh/sshd_config \
/mnt/oldserver/etc/ssh/sshd_config.d/ 2>/dev/null
```

The server was configured with:

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

# 7. Inspect UFW configuration

```
sudo grep -RniE 'ufw|iptables|nftables' \
/mnt/oldserver/etc/ 2>/dev/null | head -100
```

The outdated SSH rule was found in:

/mnt/oldserver/etc/ufw/user.rules

# 8. Repair the UFW rule

The old administrator IP:

102.90.96.127

was replaced with the current IP:

102.90.116.243

Command:

```
sudo sed -i 's/102\.90\.96\.127/102.90.116.243/g' \
/mnt/oldserver/etc/ufw/user.rules
```

Verification:

```
sudo grep -n -- '--dport 22' \
/mnt/oldserver/etc/ufw/user.rules
```

# 9. Unmount the file system

```
sudo umount /mnt/oldserver
```

# 10. Detach the volume

The repaired EBS volume was detached from the recovery instance.

# 11. Reattach the volume

The volume was reattached to the original EC2 instance.

# 12. Start the original instance

The original server was started again.

# 13. Verify SSH

SSH access was successfully restored.

# Verification

The recovered server confirmed:

EC2 health checks → 3/3 passed
SSH              → Working
UFW              → Active
SSH port 22      → Listening
Nginx            → Running
HTTP              → 200 OK

# Key Takeaway

EBS provides an important recovery mechanism for EC2.

When remote access is unavailable, the root volume can be attached to another compatible instance, allowing administrators to inspect and repair configuration files offline.

This technique is particularly useful for recovering from:

- Firewall lockouts
- SSH configuration errors
- Incorrect network configuration
- Broken service configuration
- Boot-related configuration problems
