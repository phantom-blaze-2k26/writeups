# HASBLCTF Writeup: Forum

**CTF Name:** HASBLCTF  
**Category:** Web Exploitation  
**Challenge/Task Name:** Forum  
**Difficulty:** Easy  
**Author:** Phantom Blaze  
**Target URL:** `http://34.77.68.154:10201`

## Objective
The objective of this challenge is to access the "private topics" on the forum board.

## Solution / Walkthrough

### 1. Initial Reconnaissance
Upon visiting the main page, we can inspect the source code to understand the client-side logic. The JavaScript contained within `app.js` (inline in the HTML) reveals that the site checks for an `admin` cookie to grant access to locked threads.

### 2. Finding the Secret Path
The JavaScript retrieves the admin secret by first loading `/robots.txt` and looking for the `Disallow` rule. Navigating to `/robots.txt` reveals a disallowed path: `/secret_page.html`.

### 3. Decoding the Admin Token
Visiting `/secret_page.html` presents a page indicating restricted access, but it contains a token box with a Base64 encoded string:
`c3VwZXJfc2VjcmV0X2FkbWluXzIwMjY=`

Decoding this Base64 string gives us the admin secret:
`super_secret_admin_2026`

### 4. Retrieving the Flag
The client-side script's `loadFlag()` function fetches the flag from `/flag.txt`. By setting our `admin` cookie to `super_secret_admin_2026` and directly requesting `/flag.txt`, we bypass the client-side checks and retrieve the flag.

```python
import requests

cookies = {'admin': 'super_secret_admin_2026'}
r = requests.get('http://34.77.68.154:10201/flag.txt', cookies=cookies)
print(r.text)
```

**Flag:**  
`HASBL{3v3n_4_B4by_C4n_Find_7h47}`
