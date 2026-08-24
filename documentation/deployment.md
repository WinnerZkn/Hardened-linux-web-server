## Deployment Guide

## 1. Launch EC2

An Ubuntu Server 24.04 LTS EC2 instance was deployed in AWS.

The instance was configured with:

- Public IPv4 address
- SSH key pair
- EBS root volume
- VPC networking
- Dedicated Security Group

## 2. Configure Network Access

The AWS Security Group allowed:

- TCP 22 from the administrator's public IP
- TCP 80 from the internet

## 3. Connect Using SSH

```
ssh -i "hardened-linux-web-server.pem" ubuntu@<PUBLIC_IP> 
```
## 4. Configure SSH Hardening

The SSH configuration was verified to include:

- PermitRootLogin no
- PasswordAuthentication no
- PubkeyAuthentication yes

## 5. Configure UFW

UFW was enabled with:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from <ADMIN_PUBLIC_IP> to any port 22 proto tcp
sudo ufw allow 80/tcp
sudo ufw enable
```

Verify:

```
sudo ufw status verbose
```

## 6. Install Nginx

```
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

Verify:

```
sudo systemctl status nginx --no-pager
```

## 7. Verify HTTP

```
curl -I http://localhost
```

Expected:

HTTP/1.1 200 OK

## 8. Verify SSH

```
sudo ss -tlnp | grep ':22'
```

Expected:

0.0.0.0:22
[::]:22

## Final State

The server successfully ran Ubuntu 24.04 LTS with:

- SSH key authentication
- Root SSH login disabled
- Password authentication disabled
- UFW enabled
- SSH restricted by source IP
- HTTP exposed for Nginx
- Nginx enabled and running
