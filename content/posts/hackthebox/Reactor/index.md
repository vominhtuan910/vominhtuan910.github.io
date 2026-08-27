---
date: "2026-08-24T13:49:08+07:00"
draft: false
title: "HackTheBox - Reactor walkthrough"
tags: ["HackTheBox", "Linux", "Privilege Escalation"]
summary: Walkthrough for Connected machine on HackTheBox
---

## Machine Summary

- **Name:** Reactor
- **Platform:** HackTheBox
- **OS:** Linux
- **Difficulty:** Easy
- **XP Reward:** 585
- **Key Vulnerability:**

---

After starting the machine, add the IP to the `/etc/hosts` for easier access:

```bash
echo "<MACHINE-IP> reactor.htb" | sudo tee -a /etc/hosts
```

For initial enumeration
