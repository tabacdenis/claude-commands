# CrowdSec Setup & Diagnostic

Analyze and fix the CrowdSec security engine on this VPS. Handle both fresh installs and existing installs with issues.

## Step 1: Check if CrowdSec is installed

Run: `which crowdsec && cscli version`

- If NOT installed, go to Step 2 (Fresh Install)
- If installed, go to Step 3 (Diagnostic)

## Step 2: Fresh Install

### 2.1 Install CrowdSec

```bash
curl -s https://install.crowdsec.net | sudo bash
sudo apt install -y crowdsec
```

### 2.2 Install the firewall bouncer

```bash
sudo apt install -y crowdsec-firewall-bouncer-iptables
```

### 2.3 Enroll in CrowdSec Console

Ask the user for their CrowdSec Console enrollment key. Do NOT proceed without it.

```bash
sudo cscli console enroll <KEY_PROVIDED_BY_USER>
sudo systemctl reload crowdsec
```

After enrollment, tell the user to accept the engine in the CrowdSec Console (https://app.crowdsec.net → Security Engines → Accept).

### 2.4 Continue to Step 3 for collection setup

## Step 3: Diagnostic & Fix

Run ALL of these commands in parallel and analyze results:

```bash
sudo cscli collections list
sudo cscli metrics show acquisition
sudo cscli simulation status
sudo cscli decisions list
sudo cscli alerts list 2>&1 | head -30
sudo systemctl status crowdsec
ss -tlnp | head -40
```

### 3.1 Detect running services and match with collections

Check what services are exposed:
- **NGINX** (ports 80/443): needs `crowdsecurity/nginx` collection + acquisition for `/var/log/nginx/access.log` and `/var/log/nginx/error.log` with label `type: nginx`
- **Apache** (ports 80/443): needs `crowdsecurity/apache2` collection + acquisition for apache logs with label `type: apache2`
- **SSH** (any port): needs `crowdsecurity/sshd` collection + acquisition for `/var/log/auth.log` with label `type: syslog`
- **Linux base**: needs `crowdsecurity/linux` collection
- **Postfix/Dovecot** (ports 25/465/587/143/993): needs `crowdsecurity/postfix` collection IF logs are accessible on host (skip if running in Docker with own firewall like Mailcow)

### 3.2 Install missing collections

For each missing collection:
```bash
sudo cscli collections install crowdsecurity/<name>
```

### 3.3 Fix acquisition config

Check existing acquisition files:
```bash
ls /etc/crowdsec/acquis.d/
cat /etc/crowdsec/acquis.d/*.yaml
```

For each service that needs monitoring, create or fix the acquisition file in `/etc/crowdsec/acquis.d/`. Example for nginx:

```yaml
# /etc/crowdsec/acquis.d/nginx.yaml
filenames:
  - /var/log/nginx/access.log
  - /var/log/nginx/error.log
labels:
  type: nginx
source: file
```

IMPORTANT: Make sure the log files actually exist before adding them. Check with `ls`.

### 3.4 Check for other issues

- **Simulation mode**: If `cscli simulation status` shows scenarios in simulation, disable with `sudo cscli simulation disable --all`
- **Whitelisting**: Check `Lines whitelisted` column in acquisition metrics. If high, investigate whitelist configs in `/etc/crowdsec/parsers/s02-enrich/`
- **Unparsed lines**: High unparsed ratio in acquisition metrics means parser mismatch. Verify label `type` matches the collection's expected type.

### 3.5 Reload and verify

```bash
sudo systemctl reload crowdsec
```

Wait a few seconds, then verify:
```bash
sudo cscli metrics show acquisition
```

Confirm that new log sources appear and lines are being parsed.

## Step 4: Summary

Report to the user:
1. What was installed/fixed
2. What services are now protected
3. What to check in a few hours (`sudo cscli alerts list`)
4. Any services that are NOT protected and why (e.g., Docker with own firewall)
