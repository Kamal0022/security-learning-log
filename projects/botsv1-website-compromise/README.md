# Incident Report: BOTS v1 — Website Compromise (imreallynotbatman.com)

## Summary
Between approximately 02:xx and 02:46 on 2016-08-11, the external IP 
`23.22.63.114` conducted a brute-force attack against the Joomla administrator 
login on `imreallynotbatman.com` (internal server `192.168.250.70`). 
Approximately 8.5 minutes after the last observed login attempt, the same 
source IP began repeatedly accessing a file called `agent.php`, which is not 
a standard Joomla file and is consistent with a web shell dropped after 
compromise. A Poison Ivy–themed defacement image was also served from the 
compromised server. This is a simulated attack from the Splunk BOTS v1 
training dataset (fictional organization "Wayne Corp," attacker group 
`po1s0n1vy`), analyzed for practice as a SOC/blue-team exercise.

## Environment
- Splunk Enterprise on Windows
- Dataset: Splunk BOTS v1 (attack-only), index `botsv1`
- Target: `www.imreallynotbatman.com` (internal IP `192.168.250.70`)
- Tools used: Splunk Search (SPL), `stats`, `rex`, raw event inspection

## Investigation

### 1. Scoping the activity
Searched for all events referencing the target domain: 78,683 events across 
four sourcetypes (`stream:http`, `suricata`, `fgt_utm`, `iis`). All traffic 
against the domain came from only two external IPs, which is not normal for 
a public website and was the first indicator something was wrong.

### 2. Brute-force attack — confirmed
Filtering `stream:http` traffic from `23.22.63.114` showed 411 POST requests, 
all against `/joomla/administrator/index.php`, all using the user agent 
`Python-urllib/2.7` (a scripting library, not a browser).

Pulling the raw `form_data` field from one event showed the actual login 
attempt:
**Figure 1: Raw event showing form_data with username=admin and passwd=rock**
<img width="1287" height="385" alt="1" src="https://github.com/user-attachments/assets/ef24445a-887f-4c7a-b930-a6f96f1701d9" />
username=admin&passwd=rock&...
**Figure 2: Extracted password values showing a common-password wordlist pattern**
<img width="1786" height="857" alt="2" src="https://github.com/user-attachments/assets/8b863771-ce06-4789-890e-c797ff960964" />


Extracting this across all 411 POSTs confirmed:
- **Username: always `admin`**
- **Password: a different common/weak password almost every time** 
  (`123456`, `111111`, `000000`, etc.) — consistent with a dictionary/
  wordlist attack
- **All 411 redirected (`status=303`) back to the same login page URL**, 
  meaning every observed attempt failed to authenticate — a successful login 
  normally redirects somewhere else, such as the admin dashboard, not back 
  to the login form itself

**Note:** the dataset does not contain a captured event showing a status 
other than 303 for these POSTs, so a directly observed successful login is 
not present in the data. Success is inferred from what happens next.

### 3. Web shell activity — likely successful compromise
The last brute-force POST was recorded at **02:46:51**. Starting at 
**02:55:22** (about 8.5 minutes later), the same source IP began repeatedly 
requesting `/joomla/agent.php` — a file that is not part of a standard 
Joomla installation. 194 requests to this file were observed, arriving every 
few hundred milliseconds during active bursts, which is consistent with an 
automated script polling a web shell for commands or output rather than a 
human browsing.

Most responses were empty; some returned a short encoded string, e.g.:
<34819d7b>S6g7MTlkN2M=</34819d7b>
Attempting to base64-decode this did not produce readable text, suggesting 
an additional layer of encoding or obfuscation (e.g. XOR) on top of base64, 
or a non-standard encoding scheme. Decoding it further was out of scope for 
this exercise — flagged as an open item.

### 4. Defacement artifact
A file `/poisonivy-is-coming-for-you-batman.jpeg` was served from the 
compromised server, referencing the attacker group `po1s0n1vy` named in the 
scenario. This is treated as a post-compromise defacement artifact, not as 
evidence of the intrusion method itself.

### 5. Cross-source correlation
- **Suricata** IDS logs showed the same URIs and volumes as `stream:http`, 
  including 4 events for the defacement image.
- **FortiGate (`fgt_utm`)** firewall logs showed the traffic almost entirely 
  as `passthrough` (allowed), with one `detected` event flagged on 
  2016-08-11 02:56:19 — shortly after the web shell activity began.
- **IIS** web server logs corroborated the same requests from the server side.

## Timeline
| Time (2016-08-11) | Event |
|---|---|
| ~02:xx | Brute-force login attempts begin against `/joomla/administrator/index.php` |
| 02:46:51 | Last observed brute-force POST (still failed, per redirect pattern) |
| 02:55:22 | First request to `/joomla/agent.php` — likely web shell access |
| 02:56:19 | FortiGate logs one `detected` event (only flagged event in the whole sequence) |
| Ongoing | Repeated `agent.php` polling continues over following hours |
| — | Poison Ivy defacement image served from the server |

## Indicators of Compromise
- Source IP: `23.22.63.114`
- User agent: `Python-urllib/2.7`
- Target endpoint: `/joomla/administrator/index.php`
- Suspicious file: `/joomla/agent.php` (non-standard, likely web shell)
- Defacement file: `/poisonivy-is-coming-for-you-batman.jpeg`

## Recommended Response Actions
1. Isolate `192.168.250.70` from the network pending investigation
2. Remove `agent.php` and audit the web root for other unauthorized files
3. Force a password reset on the Joomla `admin` account (and enforce a 
   stronger password policy — the account was vulnerable to a common-
   password wordlist)
4. Block `23.22.63.114` at the firewall
5. Review FortiGate rules — nearly all of this traffic was allowed 
   (`passthrough`); only one event was ever flagged
6. Check for further lateral movement or additional dropped files, since a 
   web shell is typically used for more than just defacement

## What I Learned
- Don't trust a stats table alone — the actual proof of the brute-force came 
  from reading one raw event's `form_data` field, not from aggregate counts
- HTTP 303 is a redirect, not proof of success or failure by itself — it has 
  to be interpreted by comparing the redirect target across many requests
- An unfamiliar filename on a known application (`agent.php` on Joomla) is a 
  strong lead worth chasing, not just another URL in a stats table
- Correlating timestamps across two findings (last brute-force attempt vs. 
  first web shell access) is what turns two separate observations into a 
  single, defensible narrative
- It's fine to flag something as unresolved (the encoding) rather than force 
  a conclusion the evidence doesn't support

## Limitations / Open Items
- The exact encoding used in the web shell's responses was not decoded
- No directly captured event shows a successful login (status ≠ 303); success 
  is inferred from the timing of subsequent web shell activity, not directly observed
- This dataset is a security-training simulation (Splunk BOTS v1), not a real 
  incident
  
