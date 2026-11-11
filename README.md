# Cybersecurity Journey

Hands-on cybersecurity lab work on Kali Linux (VirtualBox).

## Lab setup
- Kali Linux running in Oracle VirtualBox
- Apache, MariaDB and DVWA (Damn Vulnerable Web Application)
- Tools: nmap, Burp Suite, Firefox, Python HTTP server

## Results
- **SQL Injection:** Low (data extracted), Medium (500 error evidence), High (input properly protected)
- **Reflected XSS:** Low (script executed), Medium (script tags and onerror blocked)
- **CSRF:** Low (admin password changed through a hidden request)

## Notes
For educational use only, on my own lab machine.

## Week 2: Web Application Testing (DVWA)

**Environment:** DVWA (Damn Vulnerable Web Application) running locally on Kali Linux inside VirtualBox. Used only on my own machine for practice.

### 1. SQL Injection
- **Low:** Entered `1' OR '1'='1` in the User ID field. The page returned all user records. The application places user input directly into a SQL query without cleaning it first.
- **Medium:** The request returned a 500 Internal Server Error when the input was changed. This shows the input reached the query without proper handling.
- **High:** The same input was handled properly and did not return all records.
- **Fix:** Use parameterized (prepared) statements, so input is treated as data, not as SQL code.

### 2. Reflected XSS
- **Low:** `<script>alert('xss')</script>` executed and showed a popup.
- **Medium:** Script tags were stripped and the onerror attribute was blocked, so the script did not run.
- **Fix:** Encode output (for example, HTML-escape user input) before displaying it.

### 3. CSRF (Low)
- **What I did:** Hosted a local page with a hidden request to the DVWA password-change URL. The password changed while I was logged in, without using the form.
- **Fix:** Use anti-CSRF tokens and check the request origin.

### 4. Local Reconnaissance (127.0.0.1)
- **Command:** `nmap -sV 127.0.0.1`
- **Result:** Port 80 open (Apache httpd 2.4.68, Debian). All other scanned ports closed.



### Screenshots
- `IMG_20261024_194017_289.jpg`: DVWA home page after login.
## Week 3: Nmap on Local Machine (127.0.0.1)

**Environment:** Kali Linux inside VirtualBox. All scans were run against my own machine (127.0.0.1) for practice.

### Scans run
- **`nmap -sS 127.0.0.1` (SYN scan):** Port 80 (http) and port 3306 (mysql) open. The other 998 ports were closed. A SYN scan sends a connection request and reads the reply without completing the connection, so it is quicker and quieter than a full connect.
- **`nmap -sV 127.0.0.1` (service version):** Port 80 running Apache httpd 2.4.68 (Debian). Port 3306 identified as MariaDB, from its banner.
- **`nmap -sC -sV 127.0.0.1` (default scripts):** Confirmed the Apache default page title ("Apache2 Debian Default Page: It works") and the MariaDB version.
- **`nmap -p- 127.0.0.1` (all ports):** Ports 80, 3306, and 35091 open. Port 35091 was identified with `sudo ss -tlnp | grep 35091` as belonging to containerd, listening on localhost only.

### What I learned
- Port 3306 is a database service. DVWA uses it to store user accounts, and an open database port should normally be reachable only from trusted hosts.
- A default scan can miss services that a full port scan finds, so unknown open ports need to be checked, not ignored.
- Each unknown port can be traced to its process before deciding whether it matters.
## Week 8: SQL Injection Source Review (DVWA)

I read the DVWA source code for the SQL Injection module at Low security to see how user input reaches the database.

- **Finding:** The Low-level query inserts the user's input directly into the SQL string, between single quotes, without cleaning it first.
- **Why it matters:** Because the input is part of the SQL text, the database can read it as commands, not just data.
- **Fix:** Use a prepared statement (parameterized query). The SQL structure stays fixed, and the input is passed separately as data.

**What I learned:** The vulnerability is in how the query is built, not in the form itself. A dropdown or popup changes how input is entered, but only parameterized queries on the server stop injection.

### Screenshotweek11/IMG_20261111_214835_008.jpg 
- `05-sqli-source-low.png`: the `$query` line from the Low-level source.
## Week 11: Privilege Escalation Checks

**Goal:** Understand how aweek11/IMG_20261111_214835_008.jpg (or whatever name you low-privilege user on Linux can find paths to root, and check whether the system is exposed to known escalation bugs.

**What I did:**
1. Checked identity and rights: `whoami` returned `kali`, `id` showed the `sudo` group, and `sudo -l` showed full sudo access with my password.
2. Searched for SUID binaries: `find / -perm -4000 -type f 2>/dev/null | head -20`. Most were standard tools (`su`, `chfn`, `umount`, `ssh-keysign`, `pkexec`), plus `kismet_cap_linux_wifi`.
3. Searched for writable directories: `find / -writable -type d 2>/dev/null` (filtered). Only my home folder and its app caches were writable.
4. Checked for PwnKit (CVE-2021-4034): `dpkg -l polkitd pkexec | grep ^ii` showed version `127+really127-3kali1`, which is patched.
5. Checked a SUID binary: `ls -l /usr/bin/kismet_cap_linux_wifi` showed `-rwsr-xr--`, owned by root and group `kismet`. My user isn't in that group.

**What I learned:**
- SUID binaries are a common privilege escalation path, so unusual ones need checking.
- Sudo rights and group membership matter as much as file permissions.
- A version check against known CVEs is a quick way to rule out a common exploit.

**What I did not do:** I didn't run any exploit. All testing was on my own lab VM.

**Next:** Look up any unfamiliar SUID binary on GTFOBins, then move on to vulnerability scanning.



![Sudo rights and SUID search](week11/week11-sudo-and-suid-search.jpg)


*sudo -l, id, and SUID binary search on my Kali VM.*



![pkexec version and kismet SUID check](week11/week11-pkexec-kismet-check.jpg)


*pkexec/polkitd version 127 (patched for PwnKit) and kismet_cap_linux_wifi permissions.*
