# Security Hardening

## Security Layers

The server uses multiple security layers:

Internet
   ↓
AWS Security Group
   ↓
Network ACL
   ↓
Ubuntu UFW
   ↓
SSH / Nginx

## AWS Security Group

## SSH

Protocol: TCP
Port: 22
Source: Administrator public IP (/32)

SSH is not exposed to the entire internet.

## HTTP

Protocol: TCP
Port: 80
Source: 0.0.0.0/0

HTTP is publicly accessible because the server is intended to operate as a web server.

## UFW

Default policy:

Incoming: DENY
Outgoing: ALLOW

Final rules:

22/tcp    ALLOW    Administrator IP
80/tcp    ALLOW    Anywhere
80/tcp    ALLOW    Anywhere (v6)

Verify with:

```
sudo ufw status verbose
```

## SSH Hardening

The server uses key-based authentication:

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

## Security benefits.

- Direct root SSH login is disabled.
- Password-based brute-force attacks are reduced.
- Authentication relies on the SSH key pair.
- SSH network access is restricted by source IP.

## Defense in Depth

The project demonstrates defense in depth by combining:

1. AWS Security Groups
2. Network ACLs
3. UFW
4. SSH key authentication
5. Disabled root SSH login
6. Disabled password authentication

A failure or misconfiguration at one layer can still prevent access, which is why every layer must be understood and monitored.

## Security Improvement Opportunities

For a production environment, the following could be added:

- AWS Systems Manager Session Manager
- Elastic IP where appropriate
- HTTPS/TLS
- CloudWatch monitoring
- Centralized logging
- Automated patch management
- Vulnerability scanning
- EBS snapshots
- IAM least privilege
- Private subnets
- Application Load Balancer
