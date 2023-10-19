# Table of contents

## Bienvenida

* [CyberSec Notes](README.md)

## General

* [Pivoting](general/pivoting/README.md)
  * [Chisel](general/pivoting/chisel.md)
  * [Port Forwarding](general/pivoting/port-forwarding.md)
  * [Nmap con pivoting](general/pivoting/nmap-con-pivoting.md)
* [Buffer Overflow](general/buffer-overflow/README.md)
  * [Stack Based 32 bits \[in progress\]](general/buffer-overflow/stack-based-32-bits-in-progress/README.md)
    * [Windows SLMail 5.5](general/buffer-overflow/stack-based-32-bits-in-progress/windows-slmail-5.5.md)
    * [Linux, Vulnerable functions in C programs](general/buffer-overflow/stack-based-32-bits-in-progress/linux-vulnerable-functions-in-c-programs.md)
* [Google Dorks \[in progress\]](general/google-dorks-in-progress.md)
* [Sliver C2 \[in progress\]](general/sliver-c2-in-progress.md)
* [Denial of Service (DoS)](general/denial-of-service-dos/README.md)
  * [Low and Slow](general/denial-of-service-dos/low-and-slow.md)
* [Docker](general/docker.md)

## Network Services

* [Port 21 - FTP](network-services/port-21-ftp.md)
* [Port 22 - SSH](network-services/port-22-ssh.md)
* [Port 23 - Telnet](network-services/port-23-telnet.md)
* [Port 25 - SMTP](network-services/port-25-smtp.md)
* [Port 53 - DNS](network-services/port-53-dns/README.md)
  * [Deploy DNS Server with BIND](network-services/port-53-dns/deploy-dns-server-with-bind.md)
* [Port 80/443 - HTTP/HTTPS](network-services/port-80-443-http-https/README.md)
  * [Wordpress](network-services/port-80-443-http-https/wordpress.md)
  * [CMS Made Simple (CMSMS)](network-services/port-80-443-http-https/cms-made-simple-cmsms.md)
* [Port 88 - Kerberos](network-services/port-88-kerberos.md)
* [Port 386, 636, 3268, 3269 - LDAP](network-services/port-386-636-3268-3269-ldap.md)
* [Port 445 - SMB](network-services/port-445-smb.md)
* [Port 3128 - Squid](network-services/port-3128-squid.md)
* [Port 5985 - WinRM](network-services/port-5985-winrm.md)

## Pentesting Web

* [XML External Entity Injection(XXE)](pentesting-web/xml-external-entity-injection-xxe/README.md)
  * [Portswigger Lab #1: Retrieve Files](pentesting-web/xml-external-entity-injection-xxe/portswigger-lab-1-retrieve-files.md)
  * [Portswigger Lab #2: Perform SSRF](pentesting-web/xml-external-entity-injection-xxe/portswigger-lab-2-perform-ssrf.md)
  * [Portswigger Lab #6: Blind XXE to retrieve data via error messages](pentesting-web/xml-external-entity-injection-xxe/portswigger-lab-6-blind-xxe-to-retrieve-data-via-error-messages.md)
* [Open Redirect](pentesting-web/open-redirect.md)
* [SQL Inyection \[in progress\]](pentesting-web/sql-inyection-in-progress.md)
* [LFI](pentesting-web/lfi/README.md)
  * [Log Poisoning (Apache, SSH y SMPT)](pentesting-web/lfi/log-poisoning-apache-ssh-y-smpt.md)
* [Server Side Template Injection (SSTI)](pentesting-web/server-side-template-injection-ssti.md)
* [Padding Oracle](pentesting-web/padding-oracle.md)

## Ataques en Entornos Windows

* [MalDev](ataques-en-entornos-windows/maldev/README.md)
  * [AV Evasion](ataques-en-entornos-windows/maldev/av-evasion/README.md)
    * [Function call obfuscation](ataques-en-entornos-windows/maldev/av-evasion/function-call-obfuscation.md)
  * [Code Samples](ataques-en-entornos-windows/maldev/code-samples/README.md)
    * [Shellcode Execution C#](ataques-en-entornos-windows/maldev/code-samples/shellcode-execution-c.md)
    * [Shellcode Execution C++](ataques-en-entornos-windows/maldev/code-samples/shellcode-execution-c++.md)
    * [Stager HTTP C#](ataques-en-entornos-windows/maldev/code-samples/stager-http-c.md)
    * [Stager HTTP  C++](ataques-en-entornos-windows/maldev/code-samples/stager-http-c++.md)
    * [Process Inyection C++](ataques-en-entornos-windows/maldev/code-samples/process-inyection-c++.md)
    * [Process Inyection C#](ataques-en-entornos-windows/maldev/code-samples/process-inyection-c.md)
    * [XOR Encrypt C++](ataques-en-entornos-windows/maldev/code-samples/xor-encrypt-c++.md)
* [Directorio Activo](ataques-en-entornos-windows/directorio-activo/README.md)
  * [Spriying](ataques-en-entornos-windows/directorio-activo/spriying.md)
  * [Autenticacion Net-NTLMv2 y tipos de hashes](ataques-en-entornos-windows/directorio-activo/autenticacion-net-ntlmv2-y-tipos-de-hashes/README.md)
    * [Pass the Hash](ataques-en-entornos-windows/directorio-activo/autenticacion-net-ntlmv2-y-tipos-de-hashes/pass-the-hash.md)
    * [SMB Relay](ataques-en-entornos-windows/directorio-activo/autenticacion-net-ntlmv2-y-tipos-de-hashes/smb-relay.md)
  * [Autenticación Kerberos](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/README.md)
    * [Extensiones del protocolo Kerberos (SPNs & PACs)](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/extensiones-del-protocolo-kerberos-spns-and-pacs.md)
    * [AS\_REP Roasting](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/as\_rep-roasting.md)
    * [Kerberoasting](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/kerberoasting.md)
    * [Silver Ticket Attack](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/silver-ticket-attack.md)
    * [Golden Ticket Attack](ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/golden-ticket-attack.md)
  * [DCSync](ataques-en-entornos-windows/directorio-activo/dcsync.md)
  * [Mimikatz](ataques-en-entornos-windows/directorio-activo/mimikatz.md)
  * [BloodHound](ataques-en-entornos-windows/directorio-activo/bloodhound.md)
  * [Privilege Escalation](ataques-en-entornos-windows/directorio-activo/privilege-escalation/README.md)
    * [PS Credentials in XML format](ataques-en-entornos-windows/directorio-activo/privilege-escalation/ps-credentials-in-xml-format.md)
  * [Utils](ataques-en-entornos-windows/directorio-activo/utils.md)
* [Amsi Bypass](ataques-en-entornos-windows/amsi-bypass.md)

## Ataques en Entornos Linux

* [Privilege escalation \[in progress\]](ataques-en-entornos-linux/privilege-escalation-in-progress.md)

## Wireless Pentesting

* [Pre Connection Attacks](wireless-pentesting/pre-connection-attacks/README.md)
  * [WEP](wireless-pentesting/pre-connection-attacks/wep.md)
  * [WPA/WPA2](wireless-pentesting/pre-connection-attacks/wpa-wpa2.md)
* [Post Connection Attacks](wireless-pentesting/post-connection-attacks/README.md)
  * [ARP Spoof](wireless-pentesting/post-connection-attacks/arp-spoof.md)
* [Fake AP for Captive Portal](wireless-pentesting/fake-ap-for-captive-portal.md)
