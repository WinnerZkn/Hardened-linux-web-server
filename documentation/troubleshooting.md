# SSH Troubleshooting & Recovery

## Problem

SSH access to the EC2 instance failed with:

```text
ssh: connect to host 13.52.98.129 port 22: Connection timed out
```

## Investigation

The connection path was checked layer by layer:

| Layer | Result |
|---|---|
| SSH command | ✅ |
| SSH key | ✅ |
| EC2 public IP | ✅ |
| Security Group | ✅ |
| Route Table | ✅ |
| Internet Gateway | ✅ |
| Network ACL | ✅ |
| EC2 health checks | ✅ 3/3 |
| UFW | ❌ Root cause |

Root Cause
The server's UFW configuration contained an SSH rule allowing only the previous public IP:

-A ufw-user-input -p tcp --dport 22 -s 102.90.96.127 -j ACCEPT

The administrator's current public IP was:

102.90.116.243

Therefore UFW rejected the SSH connection.

## Recovery

SSH and Session Manager were unavailable, so the root EBS volume was recovered offline.

## Recovery steps

1. Stop the affected EC2 instance.
2. Identify its root EBS volume.
3. Launch a temporary Ubuntu recovery instance in the same Availability Zone.
4. Attach the affected EBS volume.
5. Identify the attached partition.
6. Mount the original filesystem.
7. Inspect the UFW configuration.
8. Replace the outdated SSH source IP.
9. Unmount the filesystem.
10. Detach the EBS volume.
11. Reattach it to the original EC2 instance.
12. Start the original instance.
13. Test SSH connectivity.

## Configuration Fix

The outdated IP was replaced with the current administrator IP:

```
sudo sed -i 's/102\.90\.96\.127/102.90.116.243/g' /mnt/oldserver/etc/ufw/user.rules
```

## Verification:

```
sudo grep -n -- '--dport 22' /mnt/oldserver/etc/ufw/user.rules
```

## Expected:

-A ufw-user-input -p tcp --dport 22 -s 102.90.116.243 -j ACCEPT

## Result

After the EBS recovery and UFW correction:

- SSH access was restored.
- EC2 health checks passed.
- UFW remained active.
- SSH remained restricted to the administrator's IP.
- Nginx was installed and running.
- HTTP returned 200 OK.

## Lesson

A cloud firewall allowing traffic does not guarantee that the operating system will accept it.
Troubleshooting must consider every security layer between the client and the service.
