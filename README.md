# 2two2.me
Website for 2two2.me

# `2two2.me` — Technical Consulting Website

Secure, containerized, and automated deployment of a single-page consulting site for 2two2.me using hardened NGINX inside a root Podman container.

---

## 🧱 Project Structure
```
2two2.me/ 
          ├── website/ # Static HTML/CSS site 
          ├── nginx/ # NGINX configuration (nginx.conf) 
          ├── logs/ # Mounted log output from container 
          ├── scripts/ # Automation scripts (CasaOS control, cert hook) 
          ├── letsencrypt/ # Safe user-space copy of certs (symlink-free) 
          ├── README.md # This file
```

---

## 🔐 Requirements

- Podman (v4.0+)
- Chainguard Hardened NGINX container (`cgr.dev/chainguard/nginx`)
- Let's Encrypt via Certbot (manual or automated)
- Public DNS pointing to the host (for SSL validation)
- Port 443 open via `ufw` or equivalent firewall
- `/etc/sysctl.conf` optionally updated for rootless support (or use root)

---

## ⚙️ Deployment Steps

1. **Create Let's Encrypt certs** using `certbot certonly` (manual or standalone mode)
2. **Copy certs (resolved)** to `~/letsencrypt/2two2.me/` using `scripts/renewal-hook.sh`
3. **Ensure files are `chown`ed to UID `65532` or mounted with `:U`**
4. **Run container as root** with `--network host` to avoid port 443 mapping issues
5. **Use hardened `nginx.conf`** with `pid /tmp/nginx.pid;` to bypass write issue
6. **Optional: enable `systemd` service** to start container on boot

---

## 📦 Files of Interest

- `nginx/nginx.conf`: Hardened Chainguard-compatible config (no `/run/nginx` usage)
- `podman run` command: See deployment docs or `scripts/start.sh`
- `scripts/renewal-hook.sh`: Copy certs on renewal and restart container

---

## 🛠 Troubleshooting

| Problem                              | Solution                                                                 |
|--------------------------------------|--------------------------------------------------------------------------|
| `Permission denied` on cert files    | Ensure container uses `:U` volume or `chown 65532:65532`                 |
| `bind() to 0.0.0.0:443 failed`       | Use `--network host` and run container as root                          |
| `access.log: read-only filesystem`   | Do not mount logs with `:ro`, use `:Z` or `:U`                           |
| `nginx.pid permission denied`        | Override with `pid /tmp/nginx.pid;` in `nginx.conf`                     |
| Certbot can't bind to port 80        | Temporarily stop CasaOS or other services on port 80                    |

---

## ❓ FAQ

**Q: Why use Chainguard's NGINX?**  
A: It's a distroless, minimal, hardened image designed for security and compliance.

**Q: Why not use Docker?**  
A: Podman is rootless by default and better aligns with secure container practices on Linux hosts.

**Q: Where should certs live?**  
A: Never in the repo — use `~/letsencrypt/2two2.me`, resolved (no symlinks), owned properly, and mounted into the container.

**Q: Can this be Ansible-ized?**  
A: Yes — and we're working on it. Ask and I’ll generate a full Ansible role.

---
