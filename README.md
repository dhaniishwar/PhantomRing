**Platform:** Hack The Box 

**Module:** PhantomRing from Sherlocks

**Difficulty:** Easy

**Category:** Malware Analysis

**completed Date:** 19 Sep, 2026

---
<h3 align =center> Story Time </h3>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Your organization's SOC team intercepted a suspicious binary during a routine threat hunting operation on a Linux server. The file was found in /var/tmp with an unusual name and was attempting to establish outbound connections. Initial analysis suggests this could be a post-exploitation agent. Your task is to perform static analysis on the binary to identify its capabilities, extract indicators of compromise, and understand the threat actor's infrastructure.
<br>

---
<h3 align =center> Walkthrough</h3>


---
<h3 align =center> Questions & Answers </h3>

<details>
<summary><b>Q1. What is the SHA256 hash of the malicious binary?</b></summary>
<b>Answer:</b> 2d7b1b2178f76c26893b2a56cbf9b36700235259e76b893d53817d5b66b634a5
</details>

<details>
<summary><b>Q2. What is the IP address hardcoded in the binary for C2 communication?</b></summary>
<b>Answer:</b> 192.168.56.1
</details>

<details>
<summary><b>Q3. What port does the agent connect to on the C2 server?</b></summary>
<b>Answer:</b> 4445
</details>

<details>
<summary><b>Q4. How many seconds does the agent wait before attempting to reconnect after a failed connection?</b></summary>
<b>Answer:</b> 120
</details>

<details>
<summary><b>Q5. How many different commands does the agent support? (excluding invalid commands)</b></summary>
<b>Answer:</b> 11
</details>

<details>
<summary><b>Q6. What Linux kernel interface does this malware abuse to evade EDR syscall monitoring?</b></summary>
<b>Answer:</b> io_uring
</details>

<details>
<summary><b>Q7. What file does the agent read to enumerate logged-in users?</b></summary>
<b>Answer:</b> /var/run/utmp
</details>

<details>
<summary><b>Q8. What directory does the agent scan when searching for SUID binaries for privilege escalation?</b></summary>
<b>Answer:</b> /usr/bin
</details>

<details>
<summary><b>Q9. What string does the agent search for in /proc/[pid]/maps to identify security tools using eBPF?</b></summary>
<b>Answer:</b> anon_inode:bpf-map
</details>

<details>
<summary><b>Q10. What is the full path of the first tracing file the agent attempts to disable?</b></summary>
<b>Answer:</b> /sys/kernel/debug/tracing/tracing_on
</details>

<details>
<summary><b>Q11. What procfs path does the agent read to find its own executable location before self-destruction?</b></summary>
<b>Answer:</b> /proc/self/exe
</details>

<details>
<summary><b>Q12. What command string is compared by the agent to trigger deletion of its own binary?</summary>
<b>Answer:</b> sdestruct
</details>
