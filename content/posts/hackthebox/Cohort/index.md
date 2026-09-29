---
date: "2026-08-24T11:31:58+07:00"
draft: true
title: "HackTheBox - Cohort walkthrough"
tags: ["HackTheBox", "Marimo", "Privilege Escalation", "Linux", "SSRF"]
summary: "Walkthrough for Cohort machine on HackTheBox"
---

## Machine Summary

- **Name:** Cohort
- **Platform:** HackTheBox
- **OS:** Linux
- **Difficulty:** Easy
- **XP Reward:** 585
- **Key Vulnerability:** TOCTOU Race condition vulnerability leads to local privilege escalation in `PackageKit`(CVE-2026-41651).

---

First, add the machine IP to `/etc/hosts` for easier access:

```bash
echo "<MACHINE_IP> cohort.htb" | sudo tee -a /etc/hosts
```

For initial enumeration, I use `nmap` for scanning open ports on target:

```bash
sudo nmap -sC -sV -vv -p- -T3 --open cohort.htb
```

We have 3 open ports, one for SSH and two for HTTP and HTTPS:

```text
PORT    STATE SERVICE  REASON         VERSION
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN9Ju3bTZsFozwXY1B2KIlEY4BA+RcNM57w4C5EjOw1QegUUyCJoO4TVOKfzy/9kd3WrPEj/FYKT2agja9/PM44=
|   256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH9qI0OvMyp03dAGXR0UPdxw7hjSwMR773Yb9Sne+7vD
80/tcp  open  http     syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to https://cohort.htb/
443/tcp open  ssl/http syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_http-title: Cohort Analytics
|_http-server-header: nginx/1.24.0 (Ubuntu)
| ssl-cert: Subject: commonName=cohort.htb/organizationName=Cohort Analytics
| Subject Alternative Name: DNS:cohort.htb, DNS:*.cohort.htb
| Issuer: commonName=cohort.htb/organizationName=Cohort Analytics
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-06-01T18:47:07
| Not valid after:  2126-05-08T18:47:07
| MD5:     2e50 cc1d 45e6 73fd 12c5 9e21 82f2 c0ae
| SHA-1:   7e85 23e7 63eb 6541 a236 a388 fdc5 2514 8ca9 8e8c
| SHA-256: b5a8 18c7 eb3c 1923 8381 2665 afcb 2e69 85e7 b6f4 84e2 5378 205d b746 e58c b39f
| -----BEGIN CERTIFICATE-----
| MIIDaDCCAlCgAwIBAgIUcO2V21Ijt3F8k9BT8lRtCdpBaMowDQYJKoZIhvcNAQEL
| BQAwMDEZMBcGA1UECgwQQ29ob3J0IEFuYWx5dGljczETMBEGA1UEAwwKY29ob3J0
| Lmh0YjAgFw0yNjA2MDExODQ3MDdaGA8yMTI2MDUwODE4NDcwN1owMDEZMBcGA1UE
| CgwQQ29ob3J0IEFuYWx5dGljczETMBEGA1UEAwwKY29ob3J0Lmh0YjCCASIwDQYJ
| KoZIhvcNAQEBBQADggEPADCCAQoCggEBAI3Ug35PR2gUsdEyzT9owy89VEy4FGTr
| bWvwqb0sOTXKB5Sr0kU3vG6qV6mPhSuZExCAGySU9JTQNzNit6+EHoePqUyfSp5I
| jRRMQqX9ZoFQEO3B+47fXaUBk0HwOrTKOvWyWp7u827rjnPGi5GHfzYCRtlTNEiS
| JVNcb7hHQGMuhhGuLhWQZutV91moixFPPGcKuNJ4XTzQh9Hf2+aYfcdWRGaaJRAE
| wqN1TYKsLvkh3U9MEuvWRgC1MvliLwLrofb3h49anlnzztnX0CNQq35V3wpvMnF2
| B/mobKvfaXoGk5dOYUiHtQ1RNDzGUiRF8v32XEkq9P09fEbK/9waoNsCAwEAAaN4
| MHYwHQYDVR0OBBYEFDWDIUl+ylzJHRBJ5gwUS885lzknMB8GA1UdIwQYMBaAFDWD
| IUl+ylzJHRBJ5gwUS885lzknMA8GA1UdEwEB/wQFMAMBAf8wIwYDVR0RBBwwGoIK
| Y29ob3J0Lmh0YoIMKi5jb2hvcnQuaHRiMA0GCSqGSIb3DQEBCwUAA4IBAQAzZZLv
| 8IYaAbk+wy769gS5F27BXwBDCx/a+mVXpkV1DeVqmnplcKATFfFSMvcArRTBh0nR
| cNJpFTpDCmJPritZ8sSvaai10i8wb/n67MNwSs4qdjgfQlMHurS7BkYfYOfYfL2s
| BtYvOZEMfTIU4lXN/ZqPewuxxzhh/tEEfjmeJg8X45xVAILYQkYYpRe4GS7PkC+R
| SDrTEh5mNQ2HrGI28Ku22l6n2gzz5egPL/7fiL6/6QaobwmFOICon52UcTTIc/ff
| vs+rApxVr0JDcytHyULfCTyAf+99O/lLP1Fwb8Iy1Bni55ACCMh7Q8BDuD9fPlmb
| cUEVVpToHF1kC9vP
|_-----END CERTIFICATE-----
| http-methods: 
|_  Supported Methods: GET HEAD
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

According to the `nmap` scan, we know that the website name is "Cohort Analytics", and the website is hosted on `nginx` server version 1.24.0:

![website](./pics/website.png)

I have checked the source code but nothing interesting there. On the homepage, there is a button "Open Client Insights" which leads to a `portal.html` page:

![portal](./pics/portal-html.png)

Reading the information on the page, we know that the the feature "Validate source" will read a file from passed URL and show the content of the file. Here, we can think of a SSRF vulnerability. I will create a `test.json` file on local machine:

```json
{ 
    "name": "test",
    "pass": "test"
}
```

Use `python3` to host the file on local server then pass the URL to the website:

```bash
python3 -m http.server 8000
```

Use Burp Suite proxy to analyze the request:
![burp-request](./pics/burp-request.png)

We can see that the request body contains 2 parameters `url` and `format`, send the request to Repeater and check the response:
![burp-response](./pics/burp-response.png)

The `preview` field contains the content of the file we created on local machine, so the idea here is abusing the `url` parameter to read the internal file from the server. As the notes on website, we cannot use internal or loopback IP like "127.0.0.1" or "localhost", but we can bypass easily by using `cohort.htb` or exact IP address of the machine.

The problem now is that we do not know which specific files in the server we can read, so I use `feroxbuster` to enumerate endpoints:

```bash
feroxbuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -u https://cohort.htb/ -s 200,301,302,401,403 --insecure
```

We have some endpoints here:
![feroxbuster](./pics/feroxbuster.png)

Try to pass each endpoint to Burp Suite Repeater and check the response, the 403 Forbidden one - `status` reveals interesting information:

![status-endpoint](./pics/status-endpoint.png)

Format the data from `preview` field:

```json
{
  "service": "cohort-edge",
  "status": "ok",
  "generated_by": "nginx",
  "upstreams": [
    {
      "name": "marketing",
      "host": "cohort.htb",
      "root": "/var/www/cohort"
    },
    {
      "name": "insights-api",
      "host": "cohort.htb",
      "path": "/api/",
      "target": "127.0.0.1:5000"
    },
    {
      "name": "notebooks",
      "host": "nb-1be3782a8afd3ad5.cohort.htb",
      "target": "127.0.0.1:8888",
      "note": "internal analyst workspace, not for external use"
    }
  ]
}
```

We have 3 internal upstreams here,
