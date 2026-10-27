# Linux Log Analysis, Automation & SIEM Ingestion

## Overview
I built a simulated `auth.log` containing failed SSH login attempts from several IP addresses plus legitimate logins, then used command-line tools to find the suspicious sources, automated the process with a Bash script, and loaded the results into Splunk Enterprise.

> **Note:** All log data in this project is simulated for learning purposes.

## Objectives
- Identify a brute-force pattern in authentication logs
- Extract and count source IP addresses
- Automate the analysis so it is repeatable
- Ingest results into a SIEM

## Tools Used
- macOS Terminal (Unix commands, transferable to Linux)
- Bash scripting, nano
- Splunk Enterprise (free trial, local install)

## Commands Used

| Step | Command | What it does |
|---|---|---|
| 1 | `mkdir log-analysis-lab` | Creates the working folder |
| 2 | `cd log-analysis-lab` | Moves into the folder |
| 3 | `grep Failed auth.log` | Shows every failed login line |
| 4 | `grep Failed auth.log > failed.txt` | Saves failed logins to a file |
| 5 | `cut -d " " -f 11 failed.txt` | Extracts the IP address field (field 11) |
| 6 | `cut -d " " -f 11 failed.txt \| sort \| uniq -c > findings.txt` | Sorts IPs and counts how often each appears |
| 7 | `chmod +x analyze.sh` | Makes the script executable |
| 8 | `./analyze.sh` | Runs the whole analysis |

## The Automation Script (`analyze.sh`)
```bash
#!/bin/bash
grep Failed auth.log > failed.txt
cut -d " " -f 11 failed.txt | sort | uniq -c > findings.txt
echo "Analysis complete. Check findings.txt for results."
```

## Findings
- One IP address (185.220.101.5) repeatedly failed logins against privileged account names such as root and admin, a typical brute-force signature: **one source, many failures, many usernames**.
- After expanding the simulated log, the script also surfaced several additional attacking IPs with different attempt counts. See `findings.txt` for the final counts.
- Legitimate logins from the user "charles" at 192.168.1.10 were correctly excluded because they did not match "Failed".

### grep output
![grep output] <img width="1465" height="514" alt="Screenshot 2026-10-27 at 3 16 57 AM" src="https://github.com/user-attachments/assets/89691626-7c5f-4839-9c46-fae0bc514c18" />


### findings.txt output
![findings output] <img width="1204" height="806" alt="Screenshot 2026-10-27 at 3 20 33 AM" src="https://github.com/user-attachments/assets/af4169fb-716d-4ac3-a2df-26e36978f3c5" />


### Script running
![script running] <img width="1469" height="599" alt="Screenshot 2026-10-27 at 3 25 11 AM" src="https://github.com/user-attachments/assets/03bd7b5d-0441-43c6-b322-61eb5e19d2fe" />


## SIEM Ingestion (Splunk Enterprise)
- Installed Splunk Enterprise locally and uploaded `findings.txt` through Add Data > Upload.
- Confirmed the data was searchable with `source="findings.txt"`.
- Learned that charting requires structured fields, so raw lines need field extraction (for example with `rex`) before they can be visualised.


## Current Status and Next Step
Analysis, automation, and Splunk ingestion are complete. The remaining step is finishing the field extraction so Splunk can render a bar chart of failed attempts per IP. This is the next thing I plan to work on.

## What I Learned
- A small set of Unix tools can answer a real security question quickly.
- Turning commands into a script makes analysis repeatable.
- SIEM visualisation depends on well-structured fields, so data preparation matters as much as the chart.
