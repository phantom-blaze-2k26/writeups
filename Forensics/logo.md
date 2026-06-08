# HASBLCTF Writeup: Logo

**CTF Name:** HASBLCTF  
**Category:** Forensics / Steganography  
**Challenge/Task Name:** Logo  
**Difficulty:** Easy  
**Author:** Phantom Blaze  
**Target File:** `HASBL_Logo.jpg`  

## Objective
The objective of this challenge is to discover the hidden flag concealed within a provided JPEG image file (`HASBL_Logo.jpg`).

## Solution / Walkthrough

### 1. Initial Analysis
When approaching a steganography challenge with a standard JPEG image, it is always a good practice to run through basic forensic checks:
* Extracting readable **Strings** to find hidden plain-text.
* Checking for **Appended Data** past the End-Of-File (EOF) marker (`FF D9`).
* Checking for embedded archives (e.g., hidden ZIP files) using tools like `binwalk` or Python scripts.
* Inspecting **EXIF Metadata** for hidden properties.

### 2. EXIF Metadata Inspection
Upon inspecting the EXIF metadata of the image (which can be done via `exiftool` or by using Python's `Pillow` library), we notice an anomaly in the `ImageDescription` tag.

Here is a snippet of the EXIF data extracted from the image:
```text
ResolutionUnit           : 1
ImageDescription         : SEFTQkx7VGg0bmtfMHVyX3RlYWNoM3JzX01zS3VicmFfNG5kX01yS2FkaXJ9
YCbCrPositioning         : 1
XResolution              : 1.0
YResolution              : 1.0
```

### 3. Decoding the Payload
The `ImageDescription` tag contains the string `SEFTQkx7VGg0bmtfMHVyX3RlYWNoM3JzX01zS3VicmFfNG5kX01yS2FkaXJ9`. 

This string exhibits the standard characteristics of **Base64 encoding** (alphanumeric characters, mixture of uppercase and lowercase). We can quickly decode this using a simple terminal command, an online decoder like CyberChef, or a Python one-liner:

```bash
# Using Python
python -c "import base64; print(base64.b64decode('SEFTQkx7VGg0bmtfMHVyX3RlYWNoM3JzX01zS3VicmFfNG5kX01yS2FkaXJ9').decode('utf-8'))"

# Using Bash
echo "SEFTQkx7VGg0bmtfMHVyX3RlYWNoM3JzX01zS3VicmFfNG5kX01yS2FkaXJ9" | base64 -d
```

### 4. The Flag
Decoding the Base64 string successfully reveals the hidden flag!

**Flag:**  
`HASBL{Th4nk_0ur_teach3rs_MsKubra_4nd_MrKadir}`
