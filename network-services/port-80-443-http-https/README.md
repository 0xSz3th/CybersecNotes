# Port 80/443 - HTTP/HTTPS

## Web Scanners

<pre class="language-bash"><code class="lang-bash"><strong>## WhatWeb
</strong><strong>whatweb -a 1 &#x3C;url> # stealth scan
</strong>whatweb -a 3 &#x3C;url> # agressive scan
whatweb -a 4 &#x3C;url> # heavy scan

## WebTech
# pip install webtech
webtech &#x3C;url>
</code></pre>

## WAF Detection

```bash
wafw00f <url>
```



## Fuzzing

<pre class="language-bash"><code class="lang-bash"><strong># Directory enumeration
</strong><strong>gobuster dir -u &#x3C;url> -w &#x3C;dictionary> -t &#x3C;threads>
</strong>gobuster dir -u &#x3C;url> -w &#x3C;dictionary> -t &#x3C;threads> -x html,php,xml     # fuzzear por extensión
<strong>wfuzz --hc=404 -u "http://web.com/FUZZ" -w &#x3C;dictionary> -t &#x3C;threads> 
</strong><strong>
</strong><strong># DNS Subdomain enumeration
</strong><strong>gobuster vhost -u &#x3C;url> -w &#x3C;dictionary> -t &#x3C;threads> --apend-domain
</strong>gobuster vhost -u http://wireless.com:8080/ -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -t 20
</code></pre>

