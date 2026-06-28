# Simple Web Server on AWS EC2 (Nginx)

A hands-on assignment in provisioning an EC2 instance on AWS and
hosting a basic website on Nginx.

> Note: an earlier exercise in this course covered the same workflow
> using Apache (`httpd`) instead — that was classwork, not part of
> this assignment, but the troubleshooting below was discovered while
> working through that exercise and carried straight into this one.

## Live demo

`http://18.221.26.251`

## Architecture

```
                    ┌─────────────────────────┐
   Internet         │   Security Group         │
  (port 80) ───────▶│   - SSH (22)  : My IP     │
                    │   - HTTP (80) : Anywhere   │
                    └─────────────┬────────────┘
                                  │
                        ┌─────────▼─────────┐
                        │   EC2 Instance     │
                        │   Amazon Linux     │
                        │   Nginx            │
                        │   /usr/share/      │
                        │     nginx/html/    │
                        │     index.html     │
                        └────────────────────┘
```

## Setup steps

### 1. Security group

| Rule | Type | Port | Source              |
|------|------|------|----------------------|
| 1    | SSH  | 22   | My IP                |
| 2    | HTTP | 80   | Anywhere (0.0.0.0/0) |

### 2. Launch EC2 instance

- AMI: Amazon Linux 2023
- Instance type: `t2.micro` (see [Challenges](#challenges--what-i-learned) for why not `t3.micro`)
- Key pair: created once, used for SSH access

### 3. Install and configure Nginx

```bash
sudo dnf update -y
sudo dnf install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 4. Replace the default page

```bash
sudo vim /usr/share/nginx/html/index.html
```

(Or use a heredoc over SSH to avoid paste/formatting issues with vim:)

```bash
sudo tee /usr/share/nginx/html/index.html > /dev/null << 'INNEREOF'
<paste your index.html content here>
INNEREOF
```

### 5. Verify

Visit the instance's public IP over **http://** (not https — there's
no TLS certificate configured, so port 443 isn't open and won't
respond).

Useful commands:

```bash
sudo systemctl status nginx
sudo systemctl restart nginx
sudo systemctl stop nginx
sudo nginx -t   # config syntax check
```

## Challenges & what I learned

- **SSH connected instantly for the Apache instance, then hung
  indefinitely on this one — no error, just silence.** That symptom
  (a hang rather than an immediate "Permission denied") pointed away
  from the key pair and toward the network layer. Confirmed by
  comparing `curl ifconfig.me` against the security group's SSH
  source: my public IP had changed since I'd last set "My IP" on the
  rule, so the connection was being silently dropped before it ever
  reached the instance.

- **The console's "My IP" auto-detect didn't always refresh
  reliably** when re-selecting it in the security group editor. The
  fix that actually worked was setting the source to **Custom** and
  typing the IP from `curl ifconfig.me` manually, with a `/32` suffix,
  rather than relying on the dropdown to re-detect it.

- **Takeaway:** if SSH hangs rather than fails outright, check the
  security group's source IP before suspecting the key, the instance,
  or the install itself.

## Tech stack

- AWS EC2 (Amazon Linux 2023)
- Nginx
- Plain HTML/CSS

## Author

Lewis — cloud computing student.
