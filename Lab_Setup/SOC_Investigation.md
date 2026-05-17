# Week 3 Lab: Log Analysis with CLI Tools

## Learning Outcomes
By the end of this lab, you will be able to:

- Use `grep` to search for specific patterns in log files  
- Utilize `awk` to extract and manipulate specific fields from log data  
- Employ `sed` to perform basic text transformations on log entries  
- Chain command-line tools together to perform complex log analysis tasks  
- Identify suspicious activity in web server logs  
- Create professional log analysis reports  

## Objective
Master essential Linux command-line tools (`grep`, `awk`, `sed`, `cut`, `sort`, `uniq`) to parse and analyze log files for security investigations.

## Scenario
You are a SOC analyst at SecureCorp investigating a potential web application attack. The security team has detected unusual traffic patterns to the company's web server. You have been provided with Apache web server access logs and need to analyze them to identify suspicious activity, potential attack vectors, and compromised systems.

## Prerequisites

- Linux system (Ubuntu VM from Week 2 lab, or Kali Linux)
- Basic understanding of Apache log format
- Text editor (nano, vim, or VS Code)

## Lab Duration
Approximately 2-3 hours

---

## Part 1: Understanding Apache Log Format (15 minutes)

### Step 1: Apache Combined Log Format

Apache logs use the "Combined Log Format" by default:

```
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

Example log entry:

```
192.168.1.100 - - [10/Jan/2024:13:55:36 +0000] "GET /index.html HTTP/1.1" 200 2326 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

**Field breakdown:**

| Field | Position | Description | Example |
|-------|----------|-------------|---------|
| IP Address | $1 | Client IP address | 192.168.1.100 |
| Identity | $2 | RFC 1413 identity (usually -) | - |
| Username | $3 | HTTP auth username (usually -) | - |
| Timestamp | $4 | Request timestamp | [10/Jan/2024:13:55:36 +0000] |
| Request | $5-$7 | HTTP method, path, protocol | "GET /index.html HTTP/1.1" |
| Status Code | $8 | HTTP response code | 200 |
| Size | $9 | Response size in bytes | 2326 |
| Referer | $10 | Referring URL | "-" |
| User-Agent | $11+ | Client browser/tool | "Mozilla/5.0..." |

### Step 2: Download Sample Log File

Create a working directory:

```bash
mkdir -p ~/soc-labs/week3
cd ~/soc-labs/week3
```

Download sample log file:

```bash
# Option 1: Use provided sample from repository
cp /path/to/soc-training-program/Lab-Resources/Sample-Data/apache-access.log .

# Option 2: Generate sample log (if not available)
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
```

Verify the file:

```bash
ls -lh access.log
head -5 access.log
```

---

## Part 2: Basic Log Analysis Commands (30 minutes)

### Exercise 1: Count Total Requests

**Question:** How many total requests are in the log file?

**Command:**

```bash
wc -l access.log
```

**Explanation:**
- `wc` = word count command
- `-l` = count lines
- Each line = one request

**Expected output:**

```
10000 access.log
```

**Document this:** Note the total number of requests in your report.

---

### Exercise 2: Extract IP Addresses

**Question:** How many unique IP addresses made requests?

**Step-by-step approach:**

1. Extract just the IP addresses (first field):

```bash
awk '{print $1}' access.log | head -10
```

**Explanation:**
- `awk '{print $1}'` = print first field (IP address)
- `| head -10` = show first 10 results

2. Sort the IP addresses:

```bash
awk '{print $1}' access.log | sort | head -10
```

**Explanation:**
- `sort` = arrange in alphabetical/numerical order
- This groups duplicate IPs together

3. Remove duplicates:

```bash
awk '{print $1}' access.log | sort | uniq | head -10
```

**Explanation:**
- `uniq` = remove adjacent duplicate lines
- Must be used after `sort`

4. Count unique IPs:

```bash
awk '{print $1}' access.log | sort | uniq | wc -l
```

**Alternative (more efficient):**

```bash
awk '{print $1}' access.log | sort -u | wc -l
```

**Explanation:**
- `sort -u` = sort and remove duplicates in one step

---

### Exercise 3: Find Top Talkers

**Question:** What are the top 10 most frequent IP addresses?

**Command:**

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

**Step-by-step breakdown:**

| Command piece | Purpose |
|---------------|---------|
| `awk '{print $1}'` | Extract IP addresses |
| `sort` | Sort them |
| `uniq -c` | Count occurrences of each unique IP |
| `sort -rn` | Sort by count (reverse numerical) |
| `head -10` | Show top 10 |

**Expected output:**

```
    523 192.168.1.100
    412 10.0.0.50
    387 172.16.0.25
    ...
```

**Analysis:** IPs with high request counts could indicate:
- Legitimate heavy users
- Web scrapers/bots
- Potential DDoS attacks
- Compromised systems

---

### Exercise 4: Analyze HTTP Status Codes

**Question:** How many 404 (Not Found) errors occurred?

**Command:**

```bash
grep " 404 " access.log | wc -l
```

**Explanation:**
- `grep " 404 "` = search for lines containing " 404 " (with spaces to avoid matching 4040, etc.)
- `wc -l` = count matching lines

**Better approach (using awk):**

```bash
awk '$9 == 404' access.log | wc -l
```

**Explanation:**
- `$9 == 404` = match lines where 9th field (status code) equals 404

**Analyze all status codes:**

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

**Expected output:**

```
   7523 200
   1245 404
    523 301
    234 500
    ...
```

---

### Exercise 5: Identify Suspicious URLs

**Question:** What are the top 5 requested URLs that resulted in a 404 error?

**Command:**

```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -rn | head -5
```

**Step-by-step:**

| Command piece | Purpose |
|---------------|---------|
| `$9 == 404` | Filter for 404 errors |
| `{print $7}` | Extract the URL (7th field) |
| `sort \| uniq -c` | Count occurrences |
| `sort -rn` | Sort by frequency |
| `head -5` | Top 5 |

**Expected output:**

```
     45 /admin/config.php
     32 /wp-admin/
     28 /.env
     23 /phpMyAdmin/
     19 /admin/
```

**Security Analysis:**
- `/admin/` paths = reconnaissance for admin panels
- `/.env` = searching for exposed environment files
- `/phpMyAdmin/` = looking for database management tools
- These are typical attack patterns!

---

## Part 3: Advanced Pattern Matching with grep (45 minutes)

### Exercise 6: Detect Scanning Activity

**Question:** How many requests were made by the user agent "Nikto"?

**Background:** Nikto is a web vulnerability scanner. Its presence indicates security scanning (could be authorized or malicious).

**Command:**

```bash
grep -i "nikto" access.log | wc -l
```

**Explanation:**
- `grep -i` = case-insensitive search
- Searches in the entire line (including User-Agent field)

**Extract full details:**

```bash
grep -i "nikto" access.log | head -5
```

**Get unique IPs using Nikto:**

```bash
grep -i "nikto" access.log | awk '{print $1}' | sort -u
```

---



