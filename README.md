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
<h3 align =center> Tools</h3>

<details>
<summary><b>sha256sum</b></summary>
&nbsp;&nbsp;&nbsp;&nbsp; Hashes the raw bytes of the file with SHA-256, producing a 64-character hex fingerprint that changes completely if a single byte of the file changes.
</details>

<details>
<summary><b>strings</b></summary>
&nbsp;&nbsp;&nbsp;&nbsp; Scans the file for runs of printable characters and print anything above a minimum length - 4 character by default on GNU strings. It's doesn't understand the file format at all; it just looks for text- shaped byte sequence anywhere in the file.
</details>

<details>
<summary><b>readelf</b></summary>
&nbsp;&nbsp;&nbsp;&nbsp; it is a Linux command-line tool that parses and displays detailed structural information directly from ELF (Executable and Linkable Format) binary files without executing them.
</details>

<details>
<summary><b>objdump</b></summary>
&nbsp;&nbsp;&nbsp;&nbsp; It is a GNU command-line utility used to inspect object files, executable binaries and shared libraries. Its most critical role in reverse engineering is disassembly - converting raw machine code back into human- readable assembly instructions.
</details>

---
<h3 align =center> Walkthrough</h3>

<b>Q1. What is the SHA256 hash of the malicious binary?</b>


<br>
&nbsp;&nbsp;&nbsp;&nbsp; Hashing is the step zero for any malware triage, it is unique fingerprint of the file. I ran sha256sum tool get hash, this pushes every byte of the file through the SHA-256 algorithm and prints a 64- character fingerprint.
<br>
<br>

```bash
sha256sum agent
```
<img width="579" height="46" alt="1" src="https://github.com/user-attachments/assets/62ef8a2b-7ee4-4865-895c-7fa371c1e252" />

---
<b>Q2. What is the IP address hardcoded in the binary for C2 communication?</b>

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Before we get the IP address, we get to known about the malicious file with tool file. it's reads the header behind the scenes and translates it into a clean, human- readable summary.
<br>
<br>

```bash
file agent
```

<img width="660" height="44" alt="2" src="https://github.com/user-attachments/assets/cc9f1c3e-6bb3-4921-8ff3-f1b31dbb628f" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; ELF 64-bit PIE executable format, x86-64 the CPU it targets, PIE meaning the OS can load it at a different memory address each run. The most important keyword is at the last one: "not stripped". Stripping means deleting a program's original function names before shipping it, which make the analysis harder. But this one wasn't stripping, whichh means the file still contains all of its original function. So, now we can use strings tool to get hidden text fragments.
<br>
<br>

```bash
strings agent
```

<img width="255" height="521" alt="4" src="https://github.com/user-attachments/assets/a9a231b6-3c33-4789-9326-a2185fece1b9" />
<br>
<img width="257" height="629" alt="5" src="https://github.com/user-attachments/assets/bfa1cba4-392c-4d32-9896-116ff2b79546" />



<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; we can clearly see the IP address in the strings list. we also find other useful information when we did not use the "grep". we can see the GCC version banner, it compiles only C and C++ languages. we can also see the filename "crtstuff.c" (".c" is the extension to run a C coded file). Because these we know that this was programmed in C laguage.
<br>
<br>

---
<b>Q3. What port does the agent connect to on the C2 server?</b>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; I already had the C2 address, but port number had never once shown up as a printable string anyway. So I switched to "objdump" tool, which disassembles the binary machine code back into human-readable assembly code. In C networking code, htons is the standard function used to convert a port number into network byte order before making a socket connection. -A5 and -B5 is to shows the 5 lines before and 5 lines after the match.
<br>
<br>

```bash
objdump -d agent | grep -A5 -B5 "htons"
```

<img width="652" height="320" alt="6" src="https://github.com/user-attachments/assets/46594fcd-d6d2-43f2-b679-717554b4a7e7" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; we can see the network bytes for the port right before call to htons, we found "mov $0x115d". Convert the network bytes into decimals, we will get port 4445.
<br>
<br>

---
<b>Q4. How many seconds does the agent wait before attempting to reconnect after a failed connection?</b>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Same move as before, but instead of htons use sleep keyword. Sleep is the keyword used in C to pauses program execution for a specified duration.
<br>
<br>

```bash
objdump -d agent | grep -A5 -B5 "sleep"
```

<img width="605" height="463" alt="7" src="https://github.com/user-attachments/assets/900eb031-3738-4226-aa86-60ac530a5505" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Same like before, right before calling sleep@plt, we can see the network bytes for the seconds to hold the program execution after a failed connection. Convert the network bytes into decimals, we will get 120 seconds. 
<br>

---
<b>Q5. How many different commands does the agent support? (excluding invalid commands)</b>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We saw few commands when we used strings tool, but known we going to only grep "cmd". Cmd is a clear tag for command names and commants.
<br>
<br>

```bash
strings agent | grep 'cmd'
```

<img width="211" height="169" alt="8" src="https://github.com/user-attachments/assets/80b12c77-76b8-4352-9cae-e5473ee6aae7" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; process_cmd and sanitize_cmd are helpers, not command handlers. Except this two, all eleven are commands.
<br>
<br>

---
<b>Q6. What Linux kernel interface does this malware abuse to evade EDR syscall monitoring?</b>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; I searched for the word "linux" in strings tool to find some interesting information. 
<br>
<br>

```bash
strings agent | grep -A5 -B5 'linux'
```

<img width="357" height="95" alt="9" src="https://github.com/user-attachments/assets/a59ca959-4b49-48eb-b7a1-02c919d294c9" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; When a C program calls external kernel features or shared libraries (like io_uring_queue_init). So, io_uring is a highly asynchronous I/O interface for the Linux kernel.
<br>

---
<b>Q7. What file does the agent read to enumerate logged-in users?</b>

<br>
&nbsp;&nbsp;&nbsp;&nbsp; 

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
