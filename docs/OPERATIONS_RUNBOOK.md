# Linux Server Operations — Manual Runbook

This is a **manual reference checklist**, adapted from the DMI Week 2 AWS EC2 / Ubuntu / Nginx training exercise conducted on 23 January 2026. It is not an automated monitor, a verified production procedure, or proof that every command was run exactly as written below.

## 1. Verify network and DNS

```bash
ip addr
ip route
nslookup google.com
ping -c 4 google.com
sudo ss -tulnp
sudo ufw status
```

Check interfaces, default gateway, DNS resolution, packet loss, open ports, and the host firewall state. Do not assume an inactive host firewall means AWS security-group rules are open or closed.

## 2. Confirm Nginx service and syntax

```bash
sudo systemctl status nginx
sudo systemctl is-enabled nginx
ps -ef | grep nginx
sudo nginx -t
```

Inspect status and processes. Before any configuration reload or restart, test syntax with `nginx -t`.

## 3. Test local serving and inspect access logs

```bash
curl -I http://localhost
sudo tail -n 5 /var/log/nginx/access.log
sudo tail -n 20 /var/log/nginx/error.log
```

`curl -I` and error-log review are recommended extensions; the source screenshots specifically show `curl http://localhost` and recent `access.log` entries. Distinguish HTTP success from application-level correctness.

## 4. Check resource pressure

```bash
uptime
top
df -h
sudo du -ah /var | sort -rh | head -10
```

Look at high load, memory pressure, full file systems, and large files/directories under `/var`.

## 5. Verify application files and routing

```bash
ls -la /var/www/html/
sudo nginx -t
```

Review `/etc/nginx/sites-available/default` for web root and SPA fallback configuration. Do not claim client-side routes exist based solely on an `index.html` fallback.

## 6. Restore a broken Nginx configuration

1. Record the failure and read the exact `nginx -t` error; do not restart an invalid configuration.
2. Fix the incorrect Nginx file and restore a known-good version if required.
3. Test: `sudo nginx -t`.
4. If successful, follow the approved change procedure to reload or restart: `sudo systemctl reload nginx` or `sudo systemctl restart nginx`.
5. Verify: `sudo systemctl status nginx` and an appropriate HTTP request.

**Lab example:** removing a closing brace produced `unexpected end of file, expecting "}"`; correcting the file and restarting Nginx returned it to `active (running)`.

## 7. Documentation and safety

Keep sanitized evidence of commands, outcomes, errors, corrective actions, and post-change verification. Never publish credentials, SSH private keys, cloud tokens, or exposed service secrets.