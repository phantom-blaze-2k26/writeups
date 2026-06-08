# HASBLCTF Writeup: Atlas

**CTF Name:** HASBLCTF  
**Category:** Web Exploitation  
**Challenge/Task Name:** Anatolian Atlas  
**Difficulty:** Medium  
**Author:** Phantom Blaze  
**Target URL:** `http://34.77.68.154:10202`

## Objective
The challenge presents a restaurant rating map of Turkey. The goal is to find the vulnerability and retrieve the hidden flag.

## Solution / Walkthrough

### 1. Discovery and Path Traversal
Inspecting the HTML source code reveals a comment hinting at a hidden file:
`<!-- Hint: flag_info.txt exists somewhere outside the public docs. -->`

We also observe that images (such as the map) are served through an endpoint: `/files?path=turkiye.png`. 
This endpoint is vulnerable to Path Traversal. By navigating up the directory tree using `../`, we can read the hidden file:
`GET /files?path=../../flag_info.txt`

### 2. The Instructions
Reading `flag_info.txt` provides the exact instructions to reveal the flag:
> If you can read this file, you are close.
> To reveal the flag:
> - Go to the Kayseri restaurant.
> - Set service to 3, food to 5, and hygiene to 4.
> - Write the comment exactly as: "Give me the flag!"
> - Submit the review to reveal the flag.

### 3. Exploitation
To submit a review, we first need to register an account and log in. Once authenticated, we submit the magic review to the Kayseri restaurant via a `POST` request to `/review/kayseri` with the specified parameters.

### 4. Retrieving the Flag
After the review is submitted, we query the API for the Kayseri restaurant data (`/api/restaurant/kayseri`). The server processes the specific review criteria and injects the flag into the JSON response.

**Flag:**  
`HASBL{7urkish_F00ds_4r3_D3lici0us}`
