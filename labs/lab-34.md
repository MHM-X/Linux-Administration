# Lab 34: "Tukaani": XZ LZMA Library Compromised

## Description
Description: (You can learn about Linux Libraries before starting this scenario).

The Linux shared library liblzma.so has been compromised (the real compromised XZ Utils liblzma has not been used). The liblzma.so at the path /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5 is the good one. Consider the same library liblzma.so.5.2.5 at other paths as compromised or malicious (ideally we would have used other real versions with different checksums).

Find all instances of this "malicious" liblzma library (remember, it's the same library but in different directory locations) and make it so none of the running processes use it, while the applications "webapp" and "jobapp" (both of which managed by systemd) still run properly (eg, stopping those applications is not a solution).

Test: lsof | grep liblzma.so.5 returns only the liblzma in the path: /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5

🔗 **Lab Link:** [SadServers - "Tukaani": XZ LZMA Library Compromised](https://sadservers.com/scenario/tukaani)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
