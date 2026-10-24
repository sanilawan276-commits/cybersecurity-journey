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
